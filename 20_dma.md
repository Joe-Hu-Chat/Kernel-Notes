DMA related kernel code





# dma_alloc_coherent

gfp=`GFP_KERNEL`, which means `attrs` is 0

![image-20241024153125108](./.20_dma/image-20241024153125108.png)



## dma_alloc_attrs

![image-20241025165044022](./.20_dma/image-20241025165044022.png)

### get_dma_ops

Need to set `CONFIG_DMA_OPS` in configuration (Not configured in C920 patch)

![image-20241024160846957](./.20_dma/image-20241024160846957.png)



### device.coherent_dma_mask

In `of_platform_device_create_pdata`, the `device.coherent_dma_mask` was set when registering a device.

![image-20241024155145122](./.20_dma/image-20241024155145122.png)



## dma_alloc_from_dev_coherent

![image-20241024154112362](./.20_dma/image-20241024154112362.png)

### dev_get_coherent_memory

If `dev->dma_mem` is not set, this way of DMA allocation will not be applied.

![image-20241028091557570](./.20_dma/image-20241028091557570.png)



### __dma_alloc_from_coherent

![image-20241024154258941](./.20_dma/image-20241024154258941.png)

## dma_alloc_direct

![image-20241028091859673](./.20_dma/image-20241028091859673.png)

### dma_go_direct

![image-20241028092058211](./.20_dma/image-20241028092058211.png)

## dma_direct_alloc

Several types of method of DMA allocation will be tried based on configuration and settings.

![image-20241024161642658](./.20_dma/image-20241024161642658.png)

![image-20241024161719235](./.20_dma/image-20241024161719235.png)

![image-20241025163735755](./.20_dma/image-20241025163735755.png)

![image-20241024170854575](./.20_dma/image-20241024170854575.png)

![image-20241024162152901](./.20_dma/image-20241024162152901.png)



### dev_is_dma_coherent

`CONFIG_ARCH_HAS_SYNC_DMA_FOR_DEVICE` and `CONFIG_ARCH_HAS_SYNC_DMA_FOR_CPU` should be configured for DMA writing to device and writing to CPU separately. Naturally, if these configurations are set, the device is not DMA coherent, i.e., `dev->dma_coherent` is left false in default.

![image-20241028092325332](./.20_dma/image-20241028092325332.png)

#### arch_sync_dma_for_device

![image-20241025094854973](./.20_dma/image-20241025094854973.png)

Add `CONFIG_ARCH_HAS_SYNC_DMA_FOR_DEVICE` and C920 patch

![image-20241025154048756](./.20_dma/image-20241025154048756.png)

![image-20241031102330162](./.20_dma/image-20241031102330162.png)

![image-20241031134041665](./.20_dma/image-20241031134041665.png)

![image-20241031134224174](./.20_dma/image-20241031134224174.png)

#### arch_sync_dma_for_cpu

![image-20241025095052234](./.20_dma/image-20241025095052234.png)

Add `CONFIG_ARCH_HAS_SYNC_DMA_FOR_CPU` and C920 patch

![image-20241025154105852](./.20_dma/image-20241025154105852.png)

![image-20241031133841751](./.20_dma/image-20241031133841751.png)



### 1. arch_dma_alloc

Depends on architecture, maybe even no implementation on some platforms.

![image-20241028095713630](./.20_dma/image-20241028095713630.png)



### 2. dma_direct_alloc_from_pool

![image-20241028100152092](./.20_dma/image-20241028100152092.png)



#### dma_direct_optimal_gfp_mask

![image-20241028104246464](./.20_dma/image-20241028104246464.png)

#### dma_alloc_from_pool

![image-20241028100854857](./.20_dma/image-20241028100854857.png)

Will try the pools in precedence of:

1. `atomic_pool_dma32`
2. `atomic_pool_dma`
3. `NULL`

##### dma_guess_pool

![image-20241028104401212](./.20_dma/image-20241028104401212.png)

#### __dma_alloc_from_pool

![image-20241028101144854](./.20_dma/image-20241028101144854.png)

##### gen_pool_alloc

![image-20241028101233497](./.20_dma/image-20241028101233497.png)

![image-20241028101312141](./.20_dma/image-20241028101312141.png)

###### gen_pool_alloc_algo_owner

![image-20241028101559824](./.20_dma/image-20241028101559824.png)

![image-20241029135326199](./.20_dma/image-20241029135326199.png)

##### dma_coherent_ok

`dma_coherent_ok` is what `phys_addr_ok` point to.

![image-20241028103219914](./.20_dma/image-20241028103219914.png)

###### phys_to_dma_direct

If everything is cool, `phys_to_dma_direct` may directly return the `phys` back.

![image-20241028102847691](./.20_dma/image-20241028102847691.png)

###### phys_to_dma

![image-20241028102925011](./.20_dma/image-20241028102925011.png)

![image-20241029135115697](./.20_dma/image-20241029135115697.png)



##### gen_pool_avail

To make sure the available free space in pool is not below `atomic_pool_size`.

![image-20241028103436511](./.20_dma/image-20241028103436511.png)



### 3. __dma_direct_alloc_pages

![image-20241024165538924](./.20_dma/image-20241024165538924.png)

#### dma_alloc_contiguous

![image-20241024165201233](./.20_dma/image-20241024165201233.png)

#### alloc_pages_node

![image-20241024165301613](./.20_dma/image-20241024165301613.png)

#### __aloc_pages_node

![image-20241024165407950](./.20_dma/image-20241024165407950.png)

#### __alloc_pages

![image-20241024165432736](./.20_dma/image-20241024165432736.png)

#### __alloc_pages_nodemask

![image-20241024165649023](./.20_dma/image-20241024165649023.png)

![image-20241024170036099](./.20_dma/image-20241024170036099.png)

![image-20241024165737755](./.20_dma/image-20241024165737755.png)

#### get_page_from_freelist

![image-20241024170125631](./.20_dma/image-20241024170125631.png)

![image-20241024170142878](./.20_dma/image-20241024170142878.png)

![image-20241024170157798](./.20_dma/image-20241024170157798.png)

![image-20241024170213902](./.20_dma/image-20241024170213902.png)

![image-20241024170231887](./.20_dma/image-20241024170231887.png)

![image-20241024170249632](./.20_dma/image-20241024170249632.png)

![image-20241024170309807](./.20_dma/image-20241024170309807.png)



### arch_dma_prep_coherent

![image-20241024171039522](./.20_dma/image-20241024171039522.png)



If `CONFIG_ARCH_HAS_DMA_PREP_COHERENT` is configured, a **flush** operation will be done before setting a memory space from cached to un-cached.

![image-20241025143454769](./.20_dma/image-20241025143454769.png)

The `DCACHE.CIPA` instruction will **w**rite **b**ack the pointed cache block and then **inv**alidate the cache entry. (The meaningful part of this instruction is actually the write-back operation, which is also called **flush**)

![image-20241111135054947](./.20_dma/image-20241111135054947.png)

#### DCACHE.CIPA

Consider switch to standard CMO extensions in future (Zicbom extension)

![image-20241111135500187](./.20_dma/image-20241111135500187.png)

#### CBO.FLUSH

The standard operation for this purpose should be a **flush operation**, instead of an invalidate operation. Actually, `CBO.FLUSH` instruction in `Zicbom` extension is doing the same thing as `DCACHE.CIPA`  in Thead cache sub-instruction set.

![image-20241111150107509](./.20_dma/image-20241111150107509.png)

![image-20241111150406562](./.20_dma/image-20241111150406562.png)



In mainline linux kernel code, we can see that the `arch_dma_prep_coherent` is using the `CBO.FLUSH` instruction.

![image-20241111151517305](./.20_dma/image-20241111151517305.png)

![image-20241111151705401](./.20_dma/image-20241111151705401.png)



The commit to add `zicbom` extension support:

![image-20241111151838144](./.20_dma/image-20241111151838144.png)

![image-20241111151925137](./.20_dma/image-20241111151925137.png)



### arch_dma_set_uncached

The calling of this function is only in a block that will not be compiled in if `CONFIG_ARCH_HAS_DMA_SET_UNCACHED` is not set, so the inline placeholder function is not needed.

![image-20241024171250623](./.20_dma/image-20241024171250623.png)



# dma_map_single

The streaming DMA mapping routines can be called from interrupt context.

![image-20241111105744925](./.20_dma/image-20241111105744925.png)



## dma_map_single_attrs

![image-20241111105845984](./.20_dma/image-20241111105845984.png)

## dma_map_page_attrs

![image-20241111110431116](./.20_dma/image-20241111110431116.png)

### dma_map_direct

![image-20241111110120826](./.20_dma/image-20241111110120826.png)

Normally, `get_dma_ops` will return `NULL` if not configuring `CONFIG_DMA_OPS`

![image-20241111110149450](./.20_dma/image-20241111110149450.png)

## dma_direct_map_page

![image-20241111133759120](./.20_dma/image-20241111133759120.png)



## arch_sync_dma_for_device

This is apparently architecture specific and if `CONFIG_ARCH_HAS_SYNC_DMA_FOR_DEVICE` is not set, it will be an empty function.

![image-20241111134210758](./.20_dma/image-20241111134210758.png)

![image-20241111134027975](./.20_dma/image-20241111134027975.png)

![image-20241111134045247](./.20_dma/image-20241111134045247.png)



# pool



## atomic_pool_work

![image-20241028104726649](./.20_dma/image-20241028104726649.png)



![image-20241028110649430](./.20_dma/image-20241028110649430.png)



## __dma_atomic_pool_init

![image-20241028110530732](./.20_dma/image-20241028110530732.png)



### gen_pool_create

![image-20241028110235406](./.20_dma/image-20241028110235406.png)

### gen_pool_set_algo

![image-20241028110323812](./.20_dma/image-20241028110323812.png)



### atomic_pool_expand

![image-20241029094126663](./.20_dma/image-20241029094126663.png)

![image-20241029094226575](./.20_dma/image-20241029094226575.png)



#### cma_in_zone

![image-20241029093745091](./.20_dma/image-20241029093745091.png)

Without `CONFIG_DMA_CMA` configured, which depends on `CONFIG_HAVE_DMA_CONTIGUOUS` and ` CONFIG_CMA`, `cma_in_zone` will return `false` directly.

![image-20241029093528847](./.20_dma/image-20241029093528847.png)



#### alloc_pages

![image-20241029094314235](./.20_dma/image-20241029094314235.png)

![image-20241029094355047](./.20_dma/image-20241029094355047.png)

![image-20241029094535149](./.20_dma/image-20241029094535149.png)

#### __alloc_pages

![image-20241029094607949](./.20_dma/image-20241029094607949.png)

![image-20241029094655927](./.20_dma/image-20241029094655927.png)

![image-20241029094717642](./.20_dma/image-20241029094717642.png)

![image-20241029094740307](./.20_dma/image-20241029094740307.png)

#### arch_dma_prep_coherent

![image-20241024171039522](./.20_dma/image-20241024171039522.png)

C920 specified DMA coherence functions:

![image-20241025143454769](./.20_dma/image-20241025143454769.png)

`dma_wbinv_range` just clean and invalidate the cache lines.



#### dma_common_contiguous_remap

![image-20241029134203796](./.20_dma/image-20241029134203796.png)

#### pgprot_dmacoherent

![image-20241029162124754](./.20_dma/image-20241029162124754.png)



### gen_pool_first_fit_order_align

![image-20241029135714315](./.20_dma/image-20241029135714315.png)



![image-20241029135852418](./.20_dma/image-20241029135852418.png)

#### bitmap_find_next_zero_area_off

![image-20241029135928851](./.20_dma/image-20241029135928851.png)



## atomic_pool_work_fn

![image-20241029135513664](./.20_dma/image-20241029135513664.png)



# vmalloc



## vmap

![image-20241029162530137](./.20_dma/image-20241029162530137.png)

![image-20241029133538277](./.20_dma/image-20241029133538277.png)

![image-20241029162923758](./.20_dma/image-20241029162923758.png)



![image-20241029133856785](./.20_dma/image-20241029133856785.png)



![image-20241029134006409](./.20_dma/image-20241029134006409.png)



### map_kernel_range

![image-20241030093729694](./.20_dma/image-20241030093729694.png)

### map_kernel_range_noflush

![image-20241030093913085](./.20_dma/image-20241030093913085.png)

![image-20241030093942740](./.20_dma/image-20241030093942740.png)

### vmap_p4d_range

![image-20241030094041139](./.20_dma/image-20241030094041139.png)

### vmap_pud_range

![image-20241030094122139](./.20_dma/image-20241030094122139.png)

### vmap_pmd_range

![image-20241030133414588](./.20_dma/image-20241030133414588.png)

### vmap_pte_range

![image-20241030133543134](./.20_dma/image-20241030133543134.png)

### set_pte_at

![image-20241030134740014](./.20_dma/image-20241030134740014.png)



![image-20241030134929028](./.20_dma/image-20241030134929028.png)

![image-20241030135333259](./.20_dma/image-20241030135333259.png)

![image-20241030140044868](./.20_dma/image-20241030140044868.png)

#### pte_t

![image-20241030142339162](./.20_dma/image-20241030142339162.png)

#### __pte

![image-20241030142417771](./.20_dma/image-20241030142417771.png)



#### set_pte

![image-20241030141357211](./.20_dma/image-20241030141357211.png)

Seems here should be the point where PBMT setting change page to uncached to achieve coherence with DMA.



## setup_vmalloc

![image-20241029134034470](./.20_dma/image-20241029134034470.png)



![image-20241029134112511](./.20_dma/image-20241029134112511.png)





# Contiguous Memory Allocator

![image-20241028135101322](./.20_dma/image-20241028135101322.png)

ref: https://www.marcusfolkesson.se/blog/contiguous-memory-allocator/

## Reserve CMA area

By device tree:

![image-20241029093047218](./.20_dma/image-20241029093047218.png)

By kernel parameter:

![image-20241029093222744](./.20_dma/image-20241029093222744.png)

By kernel configuration:

![image-20241029093241728](./.20_dma/image-20241029093241728.png)



# GFP

Get Free Pages

Userful GFP flag combinations:

![image-20241024162353897](./.20_dma/image-20241024162353897.png)

![image-20241024162458187](./.20_dma/image-20241024162458187.png)



![image-20241024164202495](./.20_dma/image-20241024164202495.png)

![image-20241024164016729](./.20_dma/image-20241024164016729.png)

![image-20241024164137795](./.20_dma/image-20241024164137795.png)



Plain integer GFP bitmasks:

![image-20241024162701265](./.20_dma/image-20241024162701265.png)



## keyword __force

In the context of C and C++ programming, particularly in kernel development, the `__force` keyword is often used as a type cast to indicate that a specific conversion should be enforced by the compiler. This is typically seen in the Linux kernel codebase, where it is used to cast types explicitly, ensuring that the compiler treats the value as a specific type **without any warnings or errors related to type safety**.



# pgprot



![image-20241031162106358](./.20_dma/image-20241031162106358.png)

The portion below in patch for kernel v5.10 is not compatible with C920R2S0. Check mainline kernel v6.6 (or v5.19) for corresponding C920 support.

![image-20241029151529859](./.20_dma/image-20241029151529859.png)

## pgprot_noncached

This is non-standard PBMT

![image-20241029151557081](./.20_dma/image-20241029151557081.png)



This is what the mainline kernel release v5.19:

![image-20241031155148688](./.20_dma/image-20241031155148688.png)



![image-20241101091302820](./.20_dma/image-20241101091302820.png)



![image-20241101091328802](./.20_dma/image-20241101091328802.png)

In arch/riscv/asm/errata_list.h (The errata_list means actual implementation not compatible with standard Svpbmt)

![image-20241101091413250](./.20_dma/image-20241101091413250.png)



### pgprot_val

![image-20241030142516546](./.20_dma/image-20241030142516546.png)

![image-20241029153316833](./.20_dma/image-20241029153316833.png)

### pgprot_t

![image-20241029163612515](./.20_dma/image-20241029163612515.png)



## pgprot_dmacoherent

![image-20241029162124754](./.20_dma/image-20241029162124754.png)

## dma_pgprot

![image-20241029162214786](./.20_dma/image-20241029162214786.png)



## PBMT

![image-20241029151749820](./.20_dma/image-20241029151749820.png)

![image-20241029152152236](./.20_dma/image-20241029152152236.png)

