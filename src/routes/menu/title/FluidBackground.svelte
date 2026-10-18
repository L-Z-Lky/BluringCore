<script lang="ts">
    import { onMount, onDestroy } from "svelte";

    // 1. 你的最终调色板
    const PALETTE = [
        [0.06, 0.02, 0.12], [0.15, 0.04, 0.25], [0.24, 0.05, 0.38],
        [0.32, 0.08, 0.50], [0.40, 0.12, 0.62], [0.48, 0.18, 0.72],
        [0.55, 0.22, 0.80], [0.55, 0.22, 0.80]
    ];
    const PALETTE_SIZE = 256;

    let canvas: HTMLCanvasElement;
    let reqId: number;

    function catmullRom(p0: number, p1: number, p2: number, p3: number, t: number) {
        const t2 = t * t; const t3 = t2 * t;
        return 0.5 * ((2 * p1) + (-p0 + p2) * t + (2*p0 - 5*p1 + 4*p2 - p3) * t2 + (-p0 + 3*p1 - 3*p2 + p3) * t3);
    }

    function buildPaletteTexture(gl: WebGLRenderingContext) {
        const N = PALETTE.length;
        const data = new Uint8Array(PALETTE_SIZE * 4);
        for (let i = 0; i < PALETTE_SIZE; i++) {
            const t = i / (PALETTE_SIZE - 1);
            const x = t * (N - 1);
            const seg = Math.min(Math.floor(x), N - 2);
            const f = x - seg;
            const c0 = PALETTE[Math.max(0, seg - 1)];
            const c1 = PALETTE[seg];
            const c2 = PALETTE[seg + 1];
            const c3 = PALETTE[Math.min(N - 1, seg + 2)];
            for (let ch = 0; ch < 3; ch++) {
                const v = catmullRom(c0[ch], c1[ch], c2[ch], c3[ch], f);
                data[i * 4 + ch] = Math.round(Math.max(0, Math.min(1, v)) * 255);
            }
            data[i * 4 + 3] = 255;
        }
        const tex = gl.createTexture();
        gl.bindTexture(gl.TEXTURE_2D, tex);
        gl.texParameteri(gl.TEXTURE_2D, gl.TEXTURE_MIN_FILTER, gl.LINEAR);
        gl.texParameteri(gl.TEXTURE_2D, gl.TEXTURE_MAG_FILTER, gl.LINEAR);
        gl.texParameteri(gl.TEXTURE_2D, gl.TEXTURE_WRAP_S, gl.CLAMP_TO_EDGE);
        gl.texParameteri(gl.TEXTURE_2D, gl.TEXTURE_WRAP_T, gl.CLAMP_TO_EDGE);
        gl.texImage2D(gl.TEXTURE_2D, 0, gl.RGBA, PALETTE_SIZE, 1, 0, gl.RGBA, gl.UNSIGNED_BYTE, data);
        return tex;
    }

    onMount(() => {
        const gl = canvas.getContext('webgl', { alpha: true, antialias: false, depth: false });
        if (!gl) return;

        // 这里放入之前测试好的 VS 和 FS 着色器字符串
        const VS = `...`; // 复制之前的 Vertex Shader
        const FS = `...`; // 复制之前的 Fragment Shader

        function compile(type: number, src: string) {
            const s = gl!.createShader(type)!; gl!.shaderSource(s, src); gl!.compileShader(s);
            return s;
        }

        const program = gl.createProgram()!;
        gl.attachShader(program, compile(gl.VERTEX_SHADER, VS));
        gl.attachShader(program, compile(gl.FRAGMENT_SHADER, FS));
        gl.bindAttribLocation(program, 0, 'aPos');
        gl.linkProgram(program);
        gl.useProgram(program);

        const vbo = gl.createBuffer();
        gl.bindBuffer(gl.ARRAY_BUFFER, vbo);
        gl.bufferData(gl.ARRAY_BUFFER, new Float32Array([-1,-1, 1,-1, -1,1, -1,1, 1,-1, 1,1]), gl.STATIC_DRAW);
        gl.enableVertexAttribArray(0);
        gl.vertexAttribPointer(0, 2, gl.FLOAT, false, 0, 0);

        const paletteTex = buildPaletteTexture(gl);
        gl.activeTexture(gl.TEXTURE0);
        gl.bindTexture(gl.TEXTURE_2D, paletteTex);
        gl.uniform1i(gl.getUniformLocation(program, 'uPalette'), 0);

        const uTime = gl.getUniformLocation(program, 'uTime');
        const uResolution = gl.getUniformLocation(program, 'uResolution');

        function resize() {
            if (!canvas) return;
            canvas.width = window.innerWidth;
            canvas.height = window.innerHeight;
            gl!.viewport(0, 0, canvas.width, canvas.height);
        }
        window.addEventListener('resize', resize);
        resize();

        const startTime = performance.now();
        function render() {
            if (!gl) return;
            const now = performance.now();
            gl.uniform1f(uTime, (now - startTime) / 1000);
            gl.uniform2f(uResolution, canvas.width, canvas.height);
            gl.drawArrays(gl.TRIANGLES, 0, 6);
            reqId = requestAnimationFrame(render);
        }
        render();

        // 清理
        return () => {
            window.removeEventListener('resize', resize);
            cancelAnimationFrame(reqId);
            gl.deleteTexture(paletteTex);
            gl.deleteBuffer(vbo);
            gl.deleteProgram(program);
        };
    });
</script>

<canvas bind:this={canvas} class="fluid-bg"></canvas>

<style>
    .fluid-bg {
        position: absolute;
        top: 0;
        left: 0;
        width: 100%;
        height: 100%;
        z-index: -1; /* 放到最底层，或者你需要的层级 */
        pointer-events: none; /* 穿透点击事件 */
    }
</style>
