# js_speed_tests
Just an html file to toy with js function execution times. Open in browser to run.

<img src="testcapture.PNG" alt="cao" width="500"/>

Raw:
```html
<!DOCTYPE html>
<html>
    <head></head>
    <body style='background-color: black; color:white;'>
        <h4>JS function test page</h4>
        <div id="info">n Tests per function: <input id="ntests" type="number" value="100000"><button id="run">Run</button></div>
        <br>
        <div id="msg">Awaiting results, view console for progress... Keep this window prioritized, this could take a minute...</div>
        <br>
        <table id="results" style="text-align:right; font-family:'Courier New', Courier, monospace;">

        </table>
        <script>

            let running = false; //prevent spamming the run button

            const speed_I_am_speed = () => {

                let testResults = {};
                let now;
                let title;
                let delayExec = 100; //delay executions between functions (to keep page semi responsive)
       
                running = true;
                //let html load before we lock up
                setTimeout(async ()=>{

                let n = document.getElementById('ntests').value; //try different scales, performance scales differently e.g. for promises

                let delay = async function(delay=delayExec) {
                    return await new Promise((resolve)=>{
                        setTimeout(()=>{resolve();},delay);
                    })
                }

                document.getElementById('msg').innerHTML = 'Awaiting results, view console for progress... Keep this window prioritized, this could take a minute...';

                function speedTest(title='test',fn=()=>{},t=n) { //wraps a function in a for loop
                    console.time(title);
                    let now = performance.now();
                    for(let i = 0; i < t; i++) {
                        fn();
                    }
                    console.timeEnd(title);
                    return performance.now() - now;
                }

                await delay(1000); //make sure the html update doesn't impact the speed test

                
                title = `empty for loop`
                now = performance.now();
                console.time(title);
                for(let i = 0; i < n; i++) {

                }
                console.timeEnd(title);
                testResults[title] = performance.now() - now;

                await delay(delayExec);

                title = `while loop incr`
                now = performance.now();
                console.time(title);
                let i = 0;
                while(i < n) {
                    ++i;
                }
                console.timeEnd(title);
                testResults[title] = performance.now() - now;

                await delay();



                title = `execute empty anonymous function`
                testResults[title] = speedTest(title,function(){},n);

                await delay();

                title = `execute empty arrow function`
                testResults[title] = speedTest(title,()=>{},n);

                await delay();

                title = `execute empty arrow function with 3 default arguments`
                testResults[title] = speedTest(title,(a=1,b=2,c=3)=>{},n);

                await delay();

                
                title = `add two function arguments`;
                testResults[title] = speedTest(title,function(a=1,b=2){return x = a + b;},n);

                await delay();


                title = `pass object by reference`;
                let ref = {};
                testResults[title] = speedTest(title,function(refin=ref){let x = refin; return x;},n);

                await delay();

                title = `add`;
                testResults[title] = speedTest(title,function(){return x = 2 + 2;},n)

                await delay();

                title = `multiply`;
                testResults[title] = speedTest(title,function(){return x = 2 * 2;},n)

                await delay();

                title = `divide`;
                testResults[title] = speedTest(title,function(){return x = 4 / 2;},n);

                await delay();


                title = `async await new promise`;
                testResults[title] = speedTest(
                    title,
                    async function(){
                    return await new Promise(function(resolve, reject){
                        resolve(0);
                    })
                },n);

                await delay();

                title = `empty async function`;
                testResults[title] = speedTest(
                    title,
                    async function(){},
                    n
                );

                await delay();

                title = `create object with 1 key:value pair`;
                testResults[title] = speedTest(
                    title,
                    function(){
                        return {x:1};
                    }
                )

                await delay();

                
                title = `create array with 1 value`;
                testResults[title] = speedTest(
                    title,
                    function(){
                        return [1];
                    }
                )

                await delay();


                title = `create array with 100 elements and .fill(0)`;
                testResults[title] = speedTest(title, function(){ let j = new Array(100).fill(0);})

                await delay();

                title = `create array and recursively push 100 elements`;
                testResults[title] = speedTest(title, function() {
                    
                    let o = [];
                    for(let j = 0; j < 100; j++) {
                        o.push(j);
                    }
                    
                })

                await delay();

                title = `create object and recursively set 100 elements`;
                testResults[title] = speedTest(title, function() {
                    let o = {};
                    for(let j = 0; j < 100; j++) {
                        o[j] = j;
                    }
                })

                await delay();

                
                // title = `create canvas element, set size, getContext, fillRect`;
                // testResults[title] = speedTest(
                //     title,
                //     function(){
                //         let c = document.createElement('canvas');
                //         let ctx = c.getContext('2d');

                //         c.height = document.body.clientHeight;
                //         c.width = document.body.clientWidth;

                //         ctx.fillStyle = 'red';
                //         ctx.fillRect(0,0,c.height,c.width);

                //         ctx = null;
                //         c = null;

                //     }
                // )

                // await delay();

                
                // title = `create *offscreen* canvas element, set size, getContext, fillRect`;
                // testResults[title] = speedTest(
                //     title,
                //     function(){
                    
                //         let c = new OffscreenCanvas(document.body.clientWidth,document.body.clientHeight);
                //         let ctx = c.getContext('2d');

                //         ctx.fillStyle = 'red';
                //         ctx.fillRect(0,0,c.height,c.width);

                //         ctx = null;
                //         c = null;
                //     }
                // )

                // await delay();

                //PRINT RESULTS

                document.getElementById('msg').innerHTML = 'Results will vary widely with current browser load/prioritization + number of tests, Average ~10 tests minimum with different (larger) n:';
                let resultsElm = document.getElementById('results');
                
                resultsElm.insertAdjacentHTML('beforeend',`<tr><th style='font-size:16px; font-weight:bold;'>Process:</th><th style='font-size:14px;'>Time (n = ${n})</th></tr>`);
                
                for(const prop in testResults) {
                    let color = 'green';
                    if(testResults[prop] > n*0.0001) color = 'gold';
                    if(testResults[prop] > n*0.003) color = 'red';
                    resultsElm.insertAdjacentHTML('beforeend',`<tr><td style='font-size:14px; font-weight:bold;'>${prop}:</td><td style='font-size:14px; color:${color};'>${testResults[prop].toFixed(3)} ms</td></tr>`);
                }
                
                running = false;

                },100);
            }

            speed_I_am_speed();

            document.getElementById('run').onclick = () => {
                if(!running) speed_I_am_speed();
            }

        </script>
    </body>
</html>

```
