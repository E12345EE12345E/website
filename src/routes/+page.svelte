<script lang="ts">
    let canvas: HTMLCanvasElement;
    let centerbutton: HTMLButtonElement;
    let aboutmefig: HTMLElement;
    let aboutmeimg: HTMLImageElement;
    let circles = $state(<number[][]>[]);
    let mousex = 0;
    let mousey = 0;
    let rawmousex = 0;
    let rawmousey = 0;
    let animx = 0;
    let animy = 0;
    let animspeeddefault = 0.005;
    let animtime = 0;
    let overcentertime = 0;
    let overcentermax = 0.5;
    let overcenteraccel = 0;
    let lockovercenter = false;
    let screenshakemagnitude = 0;
    let screenshaketime = 0;
    let aboutmestate = 0;
    let aboutmedrift = 0;
    const TOPBARHEIGHT = 50;
    const CIRCLENUM = 16;
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
        redrawCanvas();
    });

    function redrawCanvas() {
        canvas.width = window.innerWidth;
        canvas.height = window.innerHeight;
        const smallerdimension = Math.min(canvas.width, canvas.height);
        const context = canvas.getContext('2d')!;
        context.clearRect(0, 0, canvas.width, canvas.height);

        // Center Button
        const centerbuttonsize = 128;
        centerbutton.style.left = (canvas.width/2 - centerbuttonsize/2) + "px";
        centerbutton.style.top = (canvas.height/2 - centerbuttonsize/2 + TOPBARHEIGHT/2) + "px";

        // About Me
        const aboutmesize = smallerdimension*0.35;
        aboutmefig.style.width = aboutmesize + "px";
        aboutmeimg.style.width = aboutmesize + "px";
        aboutmefig.style.left = (canvas.width/2 - aboutmesize/2) + "px";
        aboutmefig.style.top = (canvas.height/2 - aboutmesize*0.65 + TOPBARHEIGHT/2) + "px";

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
            const N = CIRCLENUM;
            for (let i=0; i<N; i++) {
                circles.push(generateCircle(innerCircleX(W, H, R, N, i), innerCircleY(W, H, R, N, i)));
            }
            for (let i=0; i<N; i++) {
                circles.push(generateCircle(outerCircleX(W, H, R, N, i), outerCircleY(W, H, R, N, i)));
            }
            setInterval(updateCirclesPos, 15);
        }
    }

    function innerCircleX(W: number, H: number, R: number, N:number, i:number) { return W + 0.75*R*Math.cos(2*Math.PI*i/N); }
    function innerCircleY(W: number, H: number, R: number, N:number, i:number) { return TOPBARHEIGHT + H + 0.75*R*Math.sin(2*Math.PI*i/N); }
    function outerCircleX(W: number, H: number, R: number, N:number, i:number) { return W + 0.9*R*Math.cos(2*Math.PI*(i+0.5)/N); }
    function outerCircleY(W: number, H: number, R: number, N:number, i:number) { return TOPBARHEIGHT + H + 0.9*R*Math.sin(2*Math.PI*(i+0.5)/N); }

    function updateCirclesPos() {
        t++;
        let animdist = Math.sqrt((mousex-animx)*(mousex-animx)+(mousey-animy)*(mousey-animy));
        let animspeed = animspeeddefault*Math.min(1+0.04*animtime,4)*Math.max(Math.min(animdist*3, 1), 0.1);
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
        const W = canvas.width/2;
        const H = (canvas.height-TOPBARHEIGHT)/2;
        const R = 0.75*Math.min(W, H);
        const distx = W - rawmousex;
        const disty = TOPBARHEIGHT + H - rawmousey;
        overcentertime *= 0.99;
        overcentertime -= 0.001;
        if (lockovercenter) {
            overcenteraccel += 0.01+(0.01*overcentertime);
            overcentertime += 0.006 + 0.25*overcenteraccel*overcenteraccel*overcenteraccel;
            if (overcentertime >= 1 && aboutmestate == 0) {
                screenshakemagnitude = 20;
                aboutmestate = 1;
                centerbutton.remove();
            }
            if (aboutmestate == 1) {
                screenshaketime++;
                screenshakemagnitude *= 0.9;
                if (screenshakemagnitude < 0.1) {
                    screenshakemagnitude = 0;
                    aboutmestate = 2;
                    aboutmefig.style.opacity = "1";
                }
            }
            if (aboutmestate >= 1) {
                aboutmedrift = Math.min(aboutmedrift+0.001, 0.15);
            }
        } else if (distx*distx + disty*disty < R*R) {
            overcentertime += 0.006*(1-((distx*distx)/(R*R)+(disty*disty)/(R*R)));
        }
        overcentertime = Math.min(Math.max(overcentertime, 0), overcentermax);
        const offsetxconst = animx-0.5;
        const offsetyconst = animy-0.5;
        const largerdimension = Math.max(canvas.width, canvas.height);
        const magnitudeconst = (largerdimension/1920)*(1-Math.min(overcentertime, overcentermax) + aboutmedrift);
        const driftspeedmult = (aboutmestate>=1)?8:1;
        circles.forEach(c => {
            let magnitude = magnitudeconst * c[7];
            console.log(magnitude);
            let offsetx = offsetxconst + Math.sin(t*c[10]*driftspeedmult)*magnitude*0.02;
            let offsety = offsetyconst + Math.cos(t*c[10]*driftspeedmult)*magnitude*0.02;
            c[1] = c[8] + (offsetx * Math.cos(c[6]) * magnitude) + (offsety * Math.sin(c[6]) * magnitude) + (screenshakemagnitude * Math.sin(screenshaketime));
            c[2] = c[9] + (offsetx * Math.cos(c[6]+Math.PI/2) * magnitude) + (offsety * Math.sin(c[6]+Math.PI/2) * magnitude);
        });
    }

    function onpointermove(event: PointerEvent) {
        mousex = event.clientX/Math.max(canvas.width, 1);
        mousey = event.clientY/Math.max(canvas.height, 1);
        rawmousex = event.clientX;
        rawmousey = event.clientY;
    }

    function handleResize() {
        console.log("window resized");
        canvas.width = window.innerWidth;
        canvas.height = window.innerHeight;

        const W = canvas.width/2;
        const H = (canvas.height-TOPBARHEIGHT)/2;
        const R = Math.min(W, H);
        for (let i=0; i<CIRCLENUM; i++) {
            circles[i][8] = innerCircleX(W, H, R, CIRCLENUM, i);
            circles[i][9] = innerCircleY(W, H, R, CIRCLENUM, i);
        }
        for (let i=CIRCLENUM; i<CIRCLENUM*2; i++) {
            circles[i][8] = outerCircleX(W, H, R, CIRCLENUM, i);
            circles[i][9] = outerCircleY(W, H, R, CIRCLENUM, i);
        }
        updateCirclesPos();
    }

    function centerbuttonclicked(event: MouseEvent) {
        if (!lockovercenter) {
            console.log(event);
            overcentermax = 1;
            overcenteraccel = 0;
            lockovercenter = true;
        }
        centerbutton.style.opacity = "0";
    }
</script>

<svelte:window onresize={handleResize} />
<figure bind:this={aboutmefig} class="aboutme">
    <!-- svelte-ignore a11y_missing_attribute -->
    <img src="favicon.png" bind:this={aboutmeimg} class="aboutme">
    <figcaption>text that goes under the image to explain the image except i dont have an image to explain so the text is not explaining anything</figcaption>
</figure>
<!-- svelte-ignore a11y_consider_explicit_label -->
<!-- svelte-ignore a11y_missing_attribute -->
<button bind:this={centerbutton} onclick={centerbuttonclicked} {onpointermove} class="imagecontainer fadeout"><img src="favicon.png" class="notdraggable"></button>
<canvas bind:this={canvas} {onpointermove} class="bg" id="bg"></canvas>

<style>
    .aboutme {
        width: 200px;
    }
    figure.aboutme {
        position: absolute;
        margin: 0;
        opacity: 0;
        transition: opacity 2s ease-in-out;
    }
    figcaption {
        max-width: 100%;
        text-align: center;
    }
</style>