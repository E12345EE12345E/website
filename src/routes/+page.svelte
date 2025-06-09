<script lang="ts">
    let canvas: HTMLCanvasElement;
    let circles = $state(<number[][]>[]);
    const TOPBARHEIGHT = 50;

    function generateCircle(x: number, y: number) {
        return [
            255,          // color
            x,            // x
            y,            // y
            4,            // radius
            0,            // start angle
            Math.PI*2,    // end angle
            Math.random()*Math.PI*2, // direction
            Math.random(), // magnitude
            x,            // og x
            y             // og y
        ];
    }

    $effect(() => {
        canvas.width = canvas.offsetWidth;
        canvas.height = canvas.offsetHeight;
        const context = canvas.getContext('2d')!;
        context.clearRect(0, 0, canvas.width, canvas.height);

        // Render circles
        for (let i=0; i<circles.length; i++) {
            let c = circles[i];
            for (let n=i+1; n<circles.length; n++) {
                let c2 = circles[n];
                let distsqr = (c[1]-c2[1])*(c[1]-c2[1])+(c[2]-c2[2])*(c[2]-c2[2]);
                if (distsqr < 26000) {
                    let b = Math.min((26000-distsqr)*0.01, 255);
                    context.strokeStyle = "rgba(" + b + ", " + b + ", " + b + ", " + b + ")";
                    context.beginPath();
                    context.moveTo(c[1], c[2]);
                    context.lineTo(c2[1], c2[2]);
                    context.stroke();
                }
            }
        }
        circles.forEach(c => {
            context.fillStyle = "#" + c[0].toString(16) + c[0].toString(16) + c[0].toString(16) + "ff";
            context.beginPath();
            context.arc(c[1], c[2], c[3], c[4], c[5]);
            context.fill();
        });

        // Init circles
        if (circles.length == 0) {
            const W = canvas.width/2;
            const H = (canvas.height-TOPBARHEIGHT)/2;
            const R = Math.min(W, H);
            const N = 16;
            for (let i=0; i<N; i++) {
                circles.push(generateCircle(W + 0.75*R*Math.cos(2*Math.PI*i/N), TOPBARHEIGHT + H + 0.75*R*Math.sin(2*Math.PI*i/N)));
            }
            for (let i=0; i<N; i++) {
                circles.push(generateCircle(W + 0.9*R*Math.cos(2*Math.PI*(i+0.5)/N), TOPBARHEIGHT + H + 0.9*R*Math.sin(2*Math.PI*(i+0.5)/N)));
            }
        }
    });
</script>

<canvas bind:this={canvas} class="bg" id="bg"></canvas>