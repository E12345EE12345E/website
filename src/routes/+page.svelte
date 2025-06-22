<script lang="ts">
    let canvas: HTMLCanvasElement;
    let circles = $state(<number[][]>[]);
    let mousex = 0;
    let mousey = 0;
    let animx = 0;
    let animy = 0;
    let animspeeddefault = 0.005;
    let animtime = 0;
    const TOPBARHEIGHT = 50;
    let t = 0;

    function generateCircle(x: number, y: number) {
        return [
            255,          // color
            x,            // x
            y,            // y
            4,            // radius
            0,            // start angle
            Math.PI*2,    // end angle
            Math.random()*Math.PI*2, // direction
            100+100*Math.random(), // magnitude
            x,            // og x
            y,            // og y
            (0.001+0.001*Math.random())*(Math.random()>0.5?1:-1) // drift
        ];
    }

    $effect(() => {
        canvas.width = canvas.offsetWidth;
        canvas.height = canvas.offsetHeight;
        const smallerdimension = Math.min(canvas.width, canvas.height);
        const context = canvas.getContext('2d')!;
        context.clearRect(0, 0, canvas.width, canvas.height);

        // Render circles
        for (let i=0; i<circles.length; i++) {
            let c = circles[i];
            for (let n=i+1; n<circles.length; n++) {
                let c2 = circles[n];
                let distsqr = (c[1]-c2[1])*(c[1]-c2[1])+(c[2]-c2[2])*(c[2]-c2[2]);
                let maxdist = 0.075*smallerdimension*smallerdimension;
                if (distsqr < maxdist) {
                    let b = Math.min((maxdist-distsqr)*0.01, 255);
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

    function updateCirclesPos() {
        t++;
        let animdist = Math.sqrt((mousex-animx)*(mousex-animx)+(mousey-animy)*(mousey-animy));
        let animspeed = animspeeddefault*Math.min(1+0.04*animtime,4)*Math.max(Math.min(animdist*2, 1), 0.2);
        if (animdist > animspeed) {
            animtime++;
            let animangle = Math.atan2(mousey-animy,mousex-animx);
            animx += Math.cos(animangle) * animspeed;
            animy += Math.sin(animangle) * animspeed;
        } else {
            animx = mousex;
            animy = mousey;
            animtime = 0;
        }
        const offsetxconst = animx-0.5;
        const offsetyconst = animy-0.5;
        const magnitudeconst = (1);
        circles.forEach(c => {
            let magnitude = magnitudeconst * c[7];
            let offsetx = offsetxconst + Math.sin(t*c[10])*magnitude*0.02;
            let offsety = offsetyconst + Math.cos(t*c[10])*magnitude*0.02;
            c[1] = c[8] + (offsetx * Math.cos(c[6]) * magnitude) + (offsety * Math.sin(c[6]) * magnitude);
            c[2] = c[9] + (offsetx * Math.cos(c[6]+Math.PI/2) * magnitude) + (offsety * Math.sin(c[6]+Math.PI/2) * magnitude);
        })
    }

    function onpointermove(event: PointerEvent) {
        mousex = event.clientX/Math.max(canvas.width, 1);
        mousey = event.clientY/Math.max(canvas.height, 1);
    }

    setInterval(updateCirclesPos, 15);
</script>

<canvas bind:this={canvas} {onpointermove} class="bg" id="bg"></canvas>