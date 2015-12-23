``` vba
for i = 0 to m - 1
	scheduleStuff(streams[i]);
	recordEvent(streams[i], events[i]);
for i = 0 to m - 1
	scheduleStuff(streams[m+i]);
	recordEvent(streams[i], events[i];
```

Macht nicht wirklich Sinn, da Kepler und Maxwell eh nur `LOW` vs. `HIGH` unterstützen und dies damit nicht mehr wirklich was bringt.
Es macht aber Sinn, in `optimizeX(...)` die nicht `num_images`-fach parallelen Berechnungen in `HIGH` zu machen und die `num_images`-fach parallelen Berechnungen in `LOW` zu machen, um damit diesen seriellen Teil in den parallelen Teil eines anderen `optimizueX(...)` zu integrieren, damit der serielle Teil nicht zum Flaschenhals wird.

``` vba 
queue[1<<QUEUE_SIZE];
X_prime[(lnum_images - QUEUE_SIZE - 1) * 2 + 1];
X_temp[1<<QUEUE_SIZE];
Y[1<<lnum_images];
f[1<<lnum_images];
optimizeX(X,f[num_images],y[num_images],num_images);
optimizeF(X,Y,f,queueIndex);
optimizeRecursive(start,pos,lnum_images)
	if(lnum_images == QUEUE_SIZE)
		for(i = 1; i < 1<<QUEUE_SIZE; i++)
			optimizeF(Y[start + (i^1)], Y[start + i], f[i], i);
		for(l = 1; l < QUEUE_SIZE; l++)
			for(j = 0; j < 1<<(QUEUE_SIZE - 1); j++)
				optimizeX(X_temp[j], &(f[start + 2 * j]); &(Y[start + 2 * j]); QUEUE_SIZE>>1);
			for(i = 0; i < 1<<QUEUE_SIZE; i++)
				optimizeF(X_temp[(i>>l)^1], Y[start + i], f[start + i], i);
	else
		optimizeRecursive(start, 2, lnum_images - 1);
		optimizeRecursive(start + (1<<lnum_images), 1, lnum_images -1);
		for(j = 0; j < 1<<(lnum_images - QUEUE_SIZE); j++)
			for(i = 0; i < 1<<QUEUE_SIZE; i++)
				optimizeF(X_prime[(lnum_images - QUEUE_SIZE)<< ((j>>(lnum_images - QUEUE_SIZE - 1))^1)], Y[start + (j<<QUEUE_SIZE) + i], f[start + (j<<QUEUE_SIZE) + i], i);
		optimizeX(X_prime[(lnum_images - QUEUE_SIZE) * pos], &(f[start]), &(Y[start]), 1<<lnum_images);
```

``` cu
cufftHandle plan;
cufftCreate(&plan);
cufftSetAutoAllocation(plan, 0);
cufftMakePlanMany(plan, ...);
size_t worksize;
void* workarea;
cufftGetSize(plan, &worksize);
cudaMalloc(&workarea, worksize);
cufftSetWorkArea(plan, workarea);
```

Use the `workarea` pointer for multipe `plan`'s, to not need that much temporary space in global GPU memory. Required for #9, #10, #11. Part of #22.

Use `mmap()` to get a memory mapped version of the (preprocessed) input file and then use `cudaHostRegister()` to make it avaiable for GPU computing (the nice OS paging already manages the RAM as a cache for this). After that use `cudaHostUnregister()` to revoke the page lock that is no longer requiered.
