---
title:  Hill-Shading the Mandelbrot Set
name: 2021-01-29-mandelbrot-3
image: /img/mandelbrot_-0.19853719075520848_1.1001770019531256_0.00390625_100000.png
image-text: $$\text{width } 0.00391 \text{ centered at } -0.19854+i1.10018$$
---

<script
  src="https://cdnjs.cloudflare.com/ajax/libs/mathjax/2.7.0/MathJax.js?config=TeX-AMS-MML_HTMLorMML"
  type="text/javascript"></script>

I've now been using the Mandelbrot set images from my previous postings as my
Google Meet video conferencing background for a few weeks, choosing a new one
each day. It's been a fun conversation starter, and it's also indicative of what
Googlers are like that a large proportion of people in my meetings know exactly
what the image is.

I've now used up my first few batches of images. Time to find some more. Luckily
there is an infinite number of them.

And what's more I've been doing some experiments with hill-shading to make the
images reveal more of the structure, and also playing with the color pallette so
that the black of the actual Mandelbrot stands out better.

Shout out to [@AnnaThieme][29] for her [article][30] on coding up shaded relief
for maps. With the help of her artticle I added the following to my C++ coloring
[code][31]:

```c++
double hillshade(int ix, int iy) {
  // Values in the eight neighboring cells
  const double a = iterations(ix - 1, iy - 1);
  const double b = iterations(ix, iy - 1);
  const double c = iterations(ix + 1, iy - 1);
  const double d = iterations(ix - 1, iy);
  const double f = iterations(ix + 1, iy);
  const double g = iterations(ix - 1, iy + 1);
  const double h = iterations(ix, iy + 1);
  const double i = iterations(ix + 1, iy + 1);

  const double dzdx = ((c + 2 * f + i) - (a + 2 * d + g)) / (8 * KERNELSIZE);
  const double dzdy = ((g + 2 * h + i) - (a + 2 * b + c)) / (8 * KERNELSIZE);

  const double slope = atan(Z_FACTOR * sqrt(dzdx * dzdx + dzdy * dzdy));

  const double aspect = atan2(dzdy, -dzdx);
  double shade = ((cos(ZENITH) * cos(slope)) +
                  (sin(ZENITH) * sin(slope) * cos(AZIMUTH - aspect)));
  return shade < 0 ? 0 : shade;
}
```

## Here are some new interesting areas of the Mandelbrot Set I've found

### -0.7436438870371587 + i0.1318259042053119 width 5×10<sup>-13</sup>

<!-- http://localhost:3333/#-0.743643887037158704752191506114774_0.131825904205311970493132056385139_5e-13_100000 -->

[![video][2]][2]
[![image][1]][1]

### -0.18271806444477 + i0.66140756855431, width 1.45×10<sup>-11</sup>
<!-- http://localhost:3333/#-0.18271806444477842_0.6614075685543184_1.4551915228366852e-11_100000 -->

[![video][4]][4]
[![image][3]][3]

### 0.37001085813 + i0.67143543269, width 6×10<sup>-8</sup>

[![video][26]][26]
[![image][25]][25]

### -0.7345612674879727 + i0.3601896136089664, width 4.55×13<sup>-8</sup>

[![video][28]][28]
[![image][27]][27]

## 0.1488658918 + i0.6424077239, width  5×13<sup>-7</sup>

[![image][32]][32]

## -0.99920853376 + i0.30236435348, width 7.45×13<sup>-9</sup>

[![image][33]][33]

## -0.7489856 + i0.055768, width 0.000244

[![image][34]][34]

## And here are re-renderings of images from previous posts in this new style

Click the images to see at full resolution.

[![$$\text{width } 8 \text{ centered at } 0+i0$$][5]][5]

[![$$\text{width } 0.000244 \text{ centered at } -0.658448+i0.466852$$][6]][6]

[![$$\text{width } 0.000122 \text{ centered at } -0.715182+i0.2300282$$][7]][7]

[![$$\text{width } 0.000977 \text{ centered at } 0.284390+i0.013590$$][8]][8]

[![$$\text{width } 0.000977 \text{ centered at } 0.284430+i0.012732$$][9]][9]

[![$$\text{width } 4.657 \times 10^{-10} \text{ centered at } -0.13997533734+i0.992076239092$$][10]][10]

[![$$\text{width } 0.000977 \text{ centered at } -0.796186+i0.183227$$][11]][11]

[![$$\text{width } 1.19 \times 10^{-7} \text{ centered at } 0.250006+i0$$][12]][12]

[![$$\text{width } 7.45 \times 10^{-9} \text{ centered at } -1.9999117502+i0$$][13]][13]

[![$$\text{width } 0.00391 \text{ centered at } -0.19854+i1.10018$$][14]][14]

[![$$\text{width } 0.0001 \text{ centered at } -0.745263-i0.113042$$][15]][15]

[![$$\text{width } 2.5 \times 10^{-10} \text{ centered at } -0.749988802386+i0.006997251233$$][16]][16]

[![$$\text{width } 10^{-9} \text{ centered at } -1.67440967428+i0.00004716557$$][17]][17]

[![$$\text{width } 4.88 \times 10^{-13} \text{ centered at } -1.674409674652718+i0.000047165698791$$][18]][18]

[![$$\text{width } 2 \times 10^{-13} \text{ centered at } -1.674409674652720+i0.000047165698793$$][19]][19]

[![$$\text{width } 0.00000381 \text{ centered at } -1.28422516+i0.42732560$$][20]][20]

[![0.4464254440760-i0.4121418810742 2.328×10<sup>-10</sup> wide][21]][21]

[![-1.292628079931202-i0.352667377259980 7.28×10<sup>-12</sup> wide][22]][22]

[![0.4177429339294358+i0.2102828457812882 2.27×10<sup>-13</sup> wide][23]][23]

[![-0.8624892205832114+i0.21478927144381935 2.27×10<sup>-13</sup> wide][24]][24]


[1]: /img/mandelbrot_-0.743643887037158704752191506114774_0.131825904205311970493132056385139_5e-13_100000.png
[2]: /img/video_-0.743643887037158704752191506114774_0.131825904205311970493132056385139_5e-13_100000.gif
[3]: /img/mandelbrot_-0.18271806444477842_0.6614075685543184_1.4551915228366852e-11_100000.png
[4]: /img/video_-0.18271806444477842_0.6614075685543184_1.4551915228366852e-11_100000.gif
[5]: /img/mandelbrot_0_0_8_100000.png
[6]: /img/mandelbrot_-0.6584480285644533_-0.4668521881103515_0.000244140625_100000.png
[7]: /img/mandelbrot_-0.7151815414428709_-0.23002815246582042_0.0001220703125_100000.png
[8]: /img/mandelbrot_0.2843902587890624_-0.013590494791666673_0.0009765625_1000000.png
[9]: /img/mandelbrot_0.28443044026692704_-0.012731933593750003_0.0009765625_1000000.png
[10]: /img/mandelbrot_-0.13997533733903145_-0.9920762390921783_4.656612873077393e-10_1000000.png
[11]: /img/mandelbrot_-0.7961858113606771_-0.18322652180989607_0.0009765625_100000.png
[12]: /img/mandelbrot_0.250006_0_1.1920928955078125e-7_1000000.png
[13]: /img/mandelbrot_-1.9999117501972556_0_7.450580596923828e-9_1000000.png
[14]: /img/mandelbrot_-0.19853719075520848_1.1001770019531256_0.00390625_100000.png
[15]: /img/mandelbrot_-0.7452629166666667_-0.11304177083333333_0.0001_1000000.png
[16]: /img/mandelbrot_-0.7499888023860679_0.006997251233593751_2.5e-10_1000000.png
[17]: /img/mandelbrot_-1.6744096742808003_0.0000471655656125_1e-9_1000000.png
[18]: /img/mandelbrot_-1.6744096746527182_0.000047165698790702304_4.8828125e-13_10000000.png
[19]: /img/mandelbrot_-1.6744096746527197_0.0000471656987933374_2e-13_1000000.png
[20]: /img/mandelbrot_-1.2842251638571422_0.4273256043593089_0.000003814697265625_1000000.png
[21]: /img/mandelbrot_0.44642544407603696_-0.41214188107417926_2.3283064365386963e-10_1000000.png
[22]: /img/mandelbrot_-1.292628079931202_-0.35266737725997915_7.275957614183426e-12_1000000.png
[23]: /img/mandelbrot_0.4177429339294358_0.21028284578128817_2.2737367544323206e-13_1000000.png
[24]: /img/mandelbrot_-0.8624892205832114_0.21478927144381935_2.2737367544323206e-13_1000000.png
[25]: /img/mandelbrot_0.37001085813778134_0.6714354326948524_6e-8_100000.png
[26]: /img/video_0.37001085813778134_0.6714354326948524_6e-8_100000.gif
[27]: /img/mandelbrot_-0.7345612674879727_0.3601896136089664_4.547473508864641e-13_100000.png
[28]: /img/video_-0.7345612674879727_0.3601896136089664_4.547473508864641e-13_100000.gif
[29]: https://twitter.com/AnnaThieme
[30]: https://blog.datawrapper.de/shaded-relief-with-gdal-python/
[31]: https://github.com/eobrain/almondbread/blob/main/main.cc
[32]: /img/mandelbrot_0.14886589182917292_0.6424077238777159_5e-7_100000.png
[33]: /img/mandelbrot_-0.9992085337561246_0.3023643534824564_7.450580596923828e-9_100000.png
[34]: /img/mandelbrot_-0.748985544840495_0.05576807657877601_0.000244140625_100000.png