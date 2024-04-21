<script lang="ts">

    // simple vertex background using a canvas and quadtree

    import { onMount } from "svelte";

    let density = 0.0003; // one particle per 10 pixels
    const connectionDistance = 80;
    const boundsSize = 100;

    interface Point 
    {
        x: number;
        y: number;

        velocityX: number;
        velocityY: number;

        lastBoundIndex: number;
    }

    interface Vector
    {
        x: number;
        y: number;
    }

    interface Bounds
    {
        min: Vector;
        max: Vector;

        points: Point[];
    }

    const points: Point[] = [];
    const allSegments: Bounds[] = [];

    const Angles: Vector[] = [
        // top right bottom left
        { x: 1, y: 0 },
        { x: 0, y: 1 },
        { x: -1, y: 0 },
        { x: 0, y: -1 },
        // diagonals
        { x: 1, y: 1 },
        { x: -1, y: 1 },
        { x: 1, y: -1 },
        { x: -1, y: -1 }
    ]

    const MousePosition: Point = {
        x: 0,
        y: 0,
        velocityX: 0,
        velocityY: 0,
        lastBoundIndex: 0
    };

    window.addEventListener("mousemove", (event) => {
        MousePosition.x = event.clientX;
        MousePosition.y = event.clientY;

        MousePosition.velocityX = 0;
        MousePosition.velocityY = 0;

        MousePosition.lastBoundIndex = getBoundIndex(MousePosition.x, MousePosition.y);
    });

    // on tap or drag
    window.addEventListener("touchmove", (event) => {
        MousePosition.x = event.touches[0].clientX;
        MousePosition.y = event.touches[0].clientY;

        MousePosition.velocityX = 0;
        MousePosition.velocityY = 0;

        MousePosition.lastBoundIndex = getBoundIndex(MousePosition.x, MousePosition.y);
    });

    let frame = 0;
    let FPS = 0;

    const updateTime = 10

    setInterval(() => {
        //console.log(`FPS: ${frame*updateTime}`);

        lastTenFrames.push(frame * updateTime);
        if (lastTenFrames.length > 10)
        {
            lastTenFrames.shift();
        }

        FPS = frame * updateTime;
        frame = 0;
        
        // if all 10 frames are less then 60, decrease the density
        if (lastTenFrames.every((frame) => frame < 60) && lastTenFrames.length >= 5)
        {
            density -= 0.0001;
            onResize()
            console.log("Decreasing density!");
            console.log(lastTenFrames)
        }

    }, 1000 / updateTime);

    let lastTenFrames: number[] = [];

    function onAnimationFrame()
    {

        frame++;

        if (points.length == 0)
        {
            window.requestAnimationFrame(onAnimationFrame);
            return;
        }
        if (allSegments.length == 0)
        {
            window.requestAnimationFrame(onAnimationFrame);
            return;
        }

        const canvas = document.querySelector("canvas")!;
        const context = canvas!.getContext("2d")!;

        canvas.width = window.innerWidth;
        canvas.height = window.innerHeight;

        context.clearRect(0, 0, window.innerWidth, window.innerHeight);

        for (let i = 0; i < points.length; i++)
        {
            const point = points[i];

            point.x += point.velocityX;
            point.y += point.velocityY;

            if (point.x < 0 || point.x > window.innerWidth)
            {
                point.velocityX *= -1;
            }

            if (point.y < 0 || point.y > window.innerHeight)
            {
                point.velocityY *= -1;
            }

            // get all the children elements of #ContentHolder
            const children = document.querySelector("#ContentHolder")!.children;
            // loop through all the children and if the point is inside the child element, invert the velocity
            for (let i = 0; i < children.length; i++)
            {
                const child = children[i] as HTMLElement;
                // if the child element is a canvas, skip it
                if (child.classList.contains("BackgroundCanvas"))
                {
                    continue;
                }

                const rect = child.getBoundingClientRect();

                if (point.x > rect.left && point.x < rect.right && point.y > rect.top && point.y < rect.bottom)
                {
                    if (point.x < rect.left + 5 || point.x > rect.right - 5)
                    {
                        point.velocityX *= -1;
                    }
                    if (point.y < rect.top + 5 || point.y > rect.bottom - 5)
                    {
                        point.velocityY *= -1;
                    }
                } else {

                }
            }

            context.beginPath();
            context.arc(point.x, point.y, 1, 0, Math.PI * 2);
            context.fill();

            let xm = Math.max(point.x, 0);
            let ym = Math.max(point.y, 0);

            // add point to segment
            const boundIndex = getBoundIndex(xm, ym);

            if (point.lastBoundIndex != boundIndex)
            {
                // remove point from old segment
                if (point.lastBoundIndex != undefined)
                {
                    allSegments[point.lastBoundIndex].points.splice(allSegments[point.lastBoundIndex].points.indexOf(point), 1);
                }

                if (boundIndex < 0 || boundIndex >= allSegments.length)
                {
                    continue;
                }

                // add point to new segment
                allSegments[boundIndex].points.push(point);
                point.lastBoundIndex = boundIndex;
            }

            // get all points in the same segment
            const closePoints: Point[] = [
                ...allSegments[boundIndex].points
            ];

            closePoints.push(MousePosition);


            for (let i = 0; i < Angles.length; i++)
            {
                const angle = Angles[i];

                const x = point.x + angle.x * boundsSize;
                const y = point.y + angle.y * boundsSize;

                const boundIndex = getBoundIndex(x, y);

                if (boundIndex < 0 || boundIndex >= allSegments.length)
                {
                    continue;
                }

                closePoints.push(...allSegments[boundIndex].points);

            }

            for (let i = 0; i < closePoints.length; i++)
            {
                const otherPoint = closePoints[i];

                if (otherPoint == point)
                {
                    continue;
                }

                const distance = PointDistance(point, otherPoint);

                if (distance < connectionDistance)
                {

                    // change the opacity based on the distance
                    context.strokeStyle = `rgba(0, 0, 0, ${distance / connectionDistance})`;

                    // change thickness based on distance
                    context.lineWidth = 1 - distance / connectionDistance;

                    context.beginPath();
                    context.moveTo(point.x, point.y);
                    context.lineTo(otherPoint.x, otherPoint.y);
                    context.stroke();

                    context.closePath();
                }
            }

        }

        window.requestAnimationFrame(onAnimationFrame);

    }

    function withinBounds(value: number, min: number, max: number)
    {
        return value >= min && value <= max;
    }

    function PointDistance(point1: Point, point2: Point)
    {
        return Math.sqrt(Math.pow(point1.x - point2.x, 2) + Math.pow(point1.y - point2.y, 2));
    }

    onMount(() => {

        window.requestAnimationFrame(onAnimationFrame);

        onResize();

        addPoints();

    });

    function addPoints()
    {

        points.splice(0, points.length);

        let pointCount = Math.floor((window.innerWidth * window.innerHeight) * density);
        
        for (let i = 0; i < pointCount; i++)
        {
            points.push({
                x: Math.random() * window.innerWidth,
                y: Math.random() * window.innerHeight,
                velocityX: Math.random() * .5 - .25,
                velocityY: Math.random() * .5 - .25,
                lastBoundIndex: 0
            });
        }

        // check if the points are in the children elements of #ContentHolder
        const children = document.querySelector("#ContentHolder")!.children;
        for (let i = 0; i < children.length; i++)
        {
            const child = children[i] as HTMLElement;
            // if the child element is a canvas, skip it
            if (child.classList.contains("BackgroundCanvas"))
            {
                continue;
            }

            const rect = child.getBoundingClientRect();

            for (let i = 0; i < points.length; i++)
            {
                const point = points[i];

                if (point.x > rect.left && point.x < rect.right && point.y > rect.top && point.y < rect.bottom)
                {
                    // remove the point from the array
                    points.splice(i, 1);
                }
            }
        }

    }

    // on screen resize
    function onResize()
    {

        let segPerRow = Math.ceil(window.innerWidth / boundsSize);
        let segPerCol = Math.ceil(window.innerHeight / boundsSize);

        let totalSegments = segPerRow * segPerCol;

        console.log(`Making ${segPerRow} x ${segPerCol} segments`)
        console.log(`Total segments: ${totalSegments}`);

        // remove all segments
        allSegments.splice(0, allSegments.length);

        // add new segments
        for (let x = 0; x < window.innerWidth / boundsSize; x++)
        {
            for (let y = 0; y < window.innerHeight / boundsSize; y++)
            {
                allSegments.push({
                    min: { x: x * boundsSize, y: y * boundsSize },
                    max: { x: (x + 1) * boundsSize, y: (y + 1) * boundsSize },
                    points: []
                });
            }
        }

        addPoints();
        
    }

    window.addEventListener("resize", onResize);

    function getBoundIndex(x: number, y: number)
    {
        let bx = Math.floor(y / boundsSize);
        let by = Math.floor(x / boundsSize);

        let boundsPerRow = Math.ceil(window.innerHeight / boundsSize);

        let i = bx + by * boundsPerRow;
        return i;
    }

    // when the space bar is pressed casue a breakpoing
    window.addEventListener("keydown", (event) => {
        if (event.key == " ")
        {
            debugger;
        }
    });

</script>

<canvas class="BackgroundCanvas"></canvas>

<style>

    canvas {
        position: fixed;
        top: 0;
        left: 0;
        z-index: -1;
        width: 100%;
        height: 100%;

        opacity: 0.5;
    }

</style>