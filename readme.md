Structure exists at many scales. Planets are mostly spherical structures of atoms, and so are stars. Stars might live in clusters of stars, which live in galaxies. To me at least, galaxies are pretty interesting structures because they appear in all sorts of shapes (i.e. not just a spherical blob of matter). Galaxies are organized into galaxy clusters and "super"clusters. And then finally, at the largest of scales, these clusters are organized into a web-like pattern which spans the entire universe.

![](https://github.com/ricsonc/universe-with-fmm-torch/blob/main/vis.png?raw=true)

In order to verify their theories, or perhaps to estimate some model parameters, cosmologists have tried to simulate the formation of these structures, and also some relevant substructures, such as dark matter haloes around galaxy clusters. These simulations typically boil down to 1. initializing a large number of point masses in some subset of R^3, 2. computing the gravitational force each point mass exerts on all the other point masses, 3. updating the positions and velocities accordingly, 4. repeat as desired.

These simulations can involve billions of these point masses and consume centuries of CPU time. So I was curious -- can we carry out similar experiments on a commodity GPU by leveraging the power of pytorch?

Of course, the naive O(n^2) "exact" algorithm for computing all pairwise forces is intractable on this scale, so I used Andy's [fast multipole method implementation](https://github.com/andyljones/pybbfmm), which does the job in O(n). This turned out to be quite a bit trickier than expected, since my use case required periodic or toroidal boundary conditions. With lots of help from Andy, I was able to tweak his code to deal with this -- that's documented [here](https://github.com/andyljones/pybbfmm/issues/2).

Another tricky issue were the initial conditions. Knowing that the universe started out with a nearly uniform mass distribution, I naively just placed all the point masses on a grid, and perturbed them slightly, hoping that this would be sufficient. However, I was never able to reproduce any of the structures correctly. Only after a very large amount of experimentation did it occur to me to take a closer read at the literature.

As it turns out, the initialization of such simulations is a large field of study itself, and there are very sophisticated methods available. As someone who is not a cosmologist, I couldn't figure out how exactly to implement even the simplest of methods, although the general idea seems to be to use inverse FFT to generate a perterbation field such that the power spectrum follows some empirically or theoretically determined curve. Despite some attempts at following the math, I didn't succeed, so I just ended up with a hacky initialization which kind of follows the same idea, and kind of works. Based on how the final results came out, the initialization is still probably very wrong, but right enough to see interesting things.

The final simulation involves just over 4 million bodies, in 2D space instead of 3D (unfortunately, this particular implementation of multipole doesn't work great in 3D), and takes about an hour to completion. 

Disclaimer: The last time I studied physics was in high school, so you should take everything here with a lot of salt. 
