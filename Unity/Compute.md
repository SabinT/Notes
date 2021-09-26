Interesting Hash function for making infinite 3D grids:
From: https://developer.download.nvidia.com/presentations/2008/GDC/GDC08_ParticleFluids.pdf

Good explanation of spatial grid sorting here: https://wickedengine.net/2018/05/21/scalabe-gpu-fluid-simulation/

```
__device__ uint calcGridHash(int3 gridPos)
{
  const uint p1 = 73856093; // some large primes
  const uint p2 = 19349663;
  const uint p3 = 83492791;
  int n = p1*gridPos.x ^ p2*gridPos.y ^ p3*gridPos.z;
  n %= numBuckets;
  return n;
}
```

Bitonic sorting explanation:
https://poniesandlight.co.uk/reflect/bitonic_merge_sort/
