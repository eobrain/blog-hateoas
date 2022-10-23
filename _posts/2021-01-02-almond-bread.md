---
layout: post
title:  Finding Mandelbrot images to use as video conference background
categories: Programming
image: /img/mandelbrot_-0.19853719075520848_1.1001770019531256_0.00390625_10000.png
image-text: $$\text{width } 0.00391 \text{ centered at } -0.19854+i1.10018$$
---
<script
  src="https://cdnjs.cloudflare.com/ajax/libs/mathjax/2.7.0/MathJax.js?config=TeX-AMS-MML_HTMLorMML"
  type="text/javascript"></script>

Motivation
----------

More than thirty years ago an exhibition in London changed how I view the world.

It was called "Schönheit im Chaos / Frontiers of Chaos", presented by the
Goethe-Institut, the German cultural organization. I still have the exhibition
book by H.O. Peitgen, P. Richter, H. Jürgens, M. Prüfer, and D.Saupe

![Schönheit im Chaos][1]

It upended what I thought I knew about mathematics. It was astounding how much
complexity could emerge from extremely basic calculations.

I was particularly drawn to the uncanny beauty and complexity of the Mandelbrot
set, which is the complex values  $$c$$ for which the iteration $$z \gets z^2 +
c$$ does not diverge when starting with $$z = 0$$. This is so simply stated[^1],
and yet generates the images below, revealing endless variations of complexity
as you zoom deeper into the set.

[^1]: Though it is a bit more complex when the complex numbers are decomposed into their real and imaginary components: $$z \gets z^2 + c$$ expands to $$z_{re} \gets z_{re}^2 - z_{im}^2 + c_{re}$$ and  $$z_{im} \gets 2 z_{re} z_{im} + c_{im}$$

Early Difficulties
------------------

Over the years I made several attempts to generate these images myself, on the
various workstations and PCs I had available at the time. But ultimately it was
frustrating because each image took so long to compute that it was painfully
slow to explore to any depth in the Mandelbrot set.

The reason for the long compute times is that for each of the $$c$$ values of a
million or so pixels you need to iterate $$z \gets z^2 + c$$ until it is
diverging ($$\lvert z \rvert > 2$$). That can be fairly fast for pixels not in
the Mandelbrot set (displayed with a color according to how many iterations).
But for pixels that are in the Mandelbrot set (displayed as black), they will
never diverge, so you have to pick some maximum number of iterations before
giving up and deeming it to be non-diverging. The big problem is pixels that are
outside the set but very close to it: they might require a large number of
iterations before they diverge, especially at high zoom levels. To get a
reasonably accurate set boundary you need the maximum iterations threshold to be
at least 1000 at moderate zoom and 10,000 or 100,000 at higher zooms. Thus you
can easily be doing billions of complex-number calculations per image.

In the end the computers I had available at the time were just too slow.

Trying Again
------------

During the 2020 Winter Holidays I started thinking about generating Mandelbrot
sets again, for the first time in more than a decade.

I realized that one thing different was that even my now several-years-old Linux
ThinkPad laptop was vastly faster than anything I tried using before, especially
if I could use all eight cores in parallel.

The first question was what programming language to use. To get the performance
I wanted, interpreted languages like Python and JavaScript were out of the
question, and I also thought that VM languages like Java or a .Net languages
probably would not give me the control I wanted. So that left the choices:

1. **Rust** is on my to-do list of languages to learn, but it was more than I
   wanted to tackle over the Holidays. Also the memory-safety features, which
   are the big advantages of Rust, while important for large-scale programming
   are not so important for this rather modest application.
2. **Go** would have been fine, but in my past experience that libraries for
   graphics and color handling are fewer and harder to use than for C++.
3. **C++** is what I ended up choosing. Carefully written it can be faster than
   anything except possible assembly-language programming (and I was not willing
   to go down that rathole)

The only other performance option I considered was to investigate using GPU
computation -- but I decided to leave that for another time.

As a first quick prototype preserved in the now-obsolete SDL branch of [the
code][12] on Github I used the SDL graphics library to throw up a window on my
laptop and set pixels.

```c++
int iterations(int maxIterationCount, const complex<double> &c) {
  complex<double> z = 0;
  for (int i = 0; i < maxIterationCount; ++i) {
    z = z * z + c;
    if (abs(z) > 2) {
      return i;
    }
  }
  return maxIterationCount;
}
```

The [C++ code][19] takes advantage of the convenience of being able to use the
`std::complex` type which means that the code `z = z * z + c` looks pretty much
like the mathematical expression $$z \gets z^2 + c$$.

The code uses `double` which I've found allows zooming down to widths of
$$10^{-13}$$ before obvious numeric artifacts appear. A possible future
extension of the project would be to use arbitrary precision math to allow
deeper zooms.

```c++
int threadCount = thread::hardware_concurrency();

void threadWorker(const Params &params, Image *img, int mod) {
  for (int iy = mod; iy < params.imgHeight; iy += threadCount)
    for (int ix = 0; ix < params.imgWidth; ++ix) {
      img->iterations(ix, iy) = iterations(params, ix, iy);
    }
  cout << "Finished thread " << mod << endl;
}

...

  vector<thread> threads;
  for (int mod = 0; mod < threadCount; ++mod) {
    threads.emplace_back(threadWorker, params, &img, mod);
  }
  for (auto &thread : threads) {
    thread.join();
  }
```

I also made sure I was using all available processing power by running multiple
threads in parallel, which was easy because each pixel can be calculated
independently. The threads interleave which rows they are working on:

The good news was this experiment showed reasonable performance, generating
megapixel images in seconds.

![Architecture][18]

However this prototype had no UI (it had hardcoded parameters) so I needed to
figure out how to put a UI on it. What I chose was a web app that operates as follows:

1. The user opens a URL with the parameters encoded in the URL hash parameters.
2. The client JS decodes the parameters and sets the `src` attribute of the
   `<img>` tag to a URL that invokes the server JS.
3. If the image file for these parameters has not already been generated, the
   server JS invokes the C++ binary, which generates the image file and writes
   it out to the server's static files directory.
4. The server JS sends back a 302 response to the client, redirecting it to the
   the static URL for the generated image file.
5. The client JS updates its UI, and its hash parameters (enabling browser
   forward and back history to work).
6. The user chooses some parameters and clicks on somewhere in the image.
7. The client JS updates the `src` attribute of the `<img>` tag according to the
   updated parameters to update the URL that invokes the server JS. (goto step
   3)

```js
app.use(express.static('public'))

app.get('/image', (req, res) => {
  const { x, y, w, i } = req.query
  const imgPath = `/cache/mandelbrot_${x}_${y}_${w}_${i}.png`
  const imgFileName = `public${imgPath}`

  if (existsSync(imgFileName)) {
    console.log('Using existing cached ', imgFileName)
    res.redirect(imgPath)
    return
  }
  console.log('Generating ', imgFileName)
  const ls = spawn('./mandelbrot', [
    '-o', imgFileName,
    '-x', x,
    '-y', y,
    '-w', w,
    '-i', i,
    '-W', imgWidth,
    '-H', imgHeight
  ])

  ls.on('close', code => {
    console.log(`child process exited with code ${code}`)
    console.log('Generated ', imgFileName)
    res.redirect(imgPath)
  })
})
```

The [server-side JavaScript code][20] is very straightforward. It provides one
endpoint for serving the static image files and a `/image` endpoint for invoking
the C++ binary and redirecting to the generated image file:

```js
let busy = false

const setCursorMagnification = () => {
  const magnification = magnificationElement.valueAsNumber
  zoomElement.innerText = 1 << magnification
  imgElement.className = `cursor-${magnification}`
}

const setIterations = () => {
  const i = Math.pow(10, iLog10Element.valueAsNumber)
  iNewElement.innerText = i
}

const doit = () => {
  if (!window.location.hash) {
    window.location = '#0_0_8_1000'
  }
  const [x, y, w, i] = window.location.hash.substr(1).split('_')
  xElement.innerText = x
  yElement.innerText = y
  wElement.innerText = w
  iElement.innerText = i
  busy = true
  imgElement.className = 'cursor-busy'
  imgElement.setAttribute('src', `/image?x=${x}&y=${y}&w=${w}&i=${i}`)
  imgElement.onload = () => {
    setCursorMagnification()
    busy = false
  }
  imgElement.onclick = event => {
    if (busy) {
      imgElement.className = 'cursor-error'
      setTimeout(() => {
        if (busy) {
          imgElement.className = 'cursor-busy'
        }
      }, 200)
      return
    }
    const { offsetX, offsetY } = event
    const scale = w / imgWidth
    const height = scale * imgHeight
    const viewPortLeft = x - w / 2
    const viewPortTop = y - height / 2
    const newX = viewPortLeft + offsetX * scale
    const newY = viewPortTop + (imgHeight - offsetY) * scale
    const magnification = magnificationElement.valueAsNumber
    const zoom = 1 << magnification
    zoomElement.innerText = zoom
    window.location = `#${newX}_${newY}_${w / zoom}_${iNewElement.innerText}`
    doit()
  }
}

window.onload = () => {
  window.onhashchange = doit
  magnificationElement.onchange = setCursorMagnification
  iLog10Element.onchange = setIterations
  setIterations()
  doit()
}
```

The [client JavaScript code][21] has the plumbing to pass down the parameters
from the HTML to the server via the image URL query parameters. It also has a
simple mutex mechanism using the `busy` Boolean to prevent the page spawning
more than one concurrent image generation, and to modify the cursor to let the
user know the state.

Results
-------

Click any image to see in full resolution, $$1920 \times 1080$$, which is
suitable for use as a video-conferencing background.

[![$$\text{width } 8 \text{ centered at } 0+i0$$][2]][2]

[![$$\text{width } 0.000244 \text{ centered at } -0.658448+i0.466852$$][3]][3]

[![$$\text{width } 0.000122 \text{ centered at } -0.715182+i0.2300282$$][4]][4]

[![$$\text{width } 0.000977 \text{ centered at } 0.284390+i0.013590$$][5]][5]

[![$$\text{width } 0.000977 \text{ centered at } 0.284430+i0.012732$$][6]][6]

[![$$\text{width } 4.657 \times 10^{-10} \text{ centered at } -0.13997533734+i0.992076239092$$][7]][7]

[![$$\text{width } 0.000977 \text{ centered at } -0.796186+i0.183227$$][8]][8]

[![$$\text{width } 1.19 \times 10^{-7} \text{ centered at } 0.250006+i0$$][9]][9]

[![$$\text{width } 7.45 \times 10^{-9} \text{ centered at } -1.9999117502+i0$$][10]][10]

[![$$\text{width } 0.00391 \text{ centered at } -0.19854+i1.10018$$][11]][11]

[![$$\text{width } 0.0001 \text{ centered at } -0.745263-i0.113042$$][13]][13]

[![$$\text{width } 2.5 \times 10^{-10} \text{ centered at } -0.749988802386+i0.006997251233$$][14]][14]

[![$$\text{width } 10^{-9} \text{ centered at } -1.67440967428+i0.00004716557$$][15]][15]

[![$$\text{width } 4.88 \times 10^{-13} \text{ centered at } -1.674409674652718+i0.000047165698791$$][16]][16]

[![$$\text{width } 2 \times 10^{-13} \text{ centered at } -1.674409674652720+i0.000047165698793$$][17]][17]

[![$$\text{width } 0.00000381 \text{ centered at } -1.28422516+i0.42732560$$][22]][22]

[1]: /img/9783920699653-us.jpg
[2]: /img/mandelbrot_0_0_8_1000.png
[3]: /img/mandelbrot_-0.6584480285644533_-0.4668521881103515_0.000244140625_10000.png
[4]: /img/mandelbrot_-0.7151815414428709_-0.23002815246582042_0.0001220703125_10000.png
[5]: /img/mandelbrot_0.2843902587890624_-0.013590494791666673_0.0009765625_100000.png
[6]: /img/mandelbrot_0.28443044026692704_-0.012731933593750003_0.0009765625_100000.png
[7]: /img/mandelbrot_-0.13997533733903145_-0.9920762390921783_4.656612873077393e-10_100000.png
[8]: /img/mandelbrot_-0.7961858113606771_-0.18322652180989607_0.0009765625_10000.png
[9]: /img/mandelbrot_0.250006_0_1.1920928955078125e-7_100000.png
[10]: /img/mandelbrot_-1.9999117501972556_0_7.450580596923828e-9_100000.png
[11]: /img/mandelbrot_-0.19853719075520848_1.1001770019531256_0.00390625_10000.png

[12]: https://github.com/eobrain/almondbread

[13]: /img/mandelbrot_-0.7452629166666667_-0.11304177083333333_0.0001_100000.png
[14]: /img/mandelbrot_-0.7499888023860679_0.006997251233593751_2.5e-10_100000.png
[15]: /img/mandelbrot_-1.6744096742808003_0.0000471655656125_1e-9_100000.png
[16]: /img/mandelbrot_-1.6744096746527182_0.000047165698790702304_4.8828125e-13_1000000.png
[17]: /img/mandelbrot_-1.6744096746527197_0.0000471656987933374_2e-13_100000.png

[18]: /img/Almondbread.svg

[19]: https://github.com/eobrain/almondbread/blob/main/main.cc
[20]: https://github.com/eobrain/almondbread/blob/main/server.js
[21]: https://github.com/eobrain/almondbread/blob/main/public/client.js

[22]: /img/mandelbrot_-1.2842251638571422_0.4273256043593089_0.000003814697265625_100000.png
