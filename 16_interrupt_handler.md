1. How are all the interrupt handlers working in Linux kernel?



# request_per_irq

![image-20240731170748338](./.16_interrupt_handler/image-20240731170748338.png)

## __request_percpu_irq

![image-20240809151428539](./.16_interrupt_handler/image-20240809151428539.png)

![image-20240801090338388](./.16_interrupt_handler/image-20240801090338388.png)

## __setup_irq

![image-20240801090710579](./.16_interrupt_handler/image-20240801090710579.png)

![image-20240801090801982](./.16_interrupt_handler/image-20240801090801982.png)

![image-20240801090823672](./.16_interrupt_handler/image-20240801090823672.png)

![image-20240801090843121](./.16_interrupt_handler/image-20240801090843121.png)

![image-20240801090914992](./.16_interrupt_handler/image-20240801090914992.png)

![image-20240801090939647](./.16_interrupt_handler/image-20240801090939647.png)

![image-20240801091010549](./.16_interrupt_handler/image-20240801091010549.png)

![image-20240801091027552](./.16_interrupt_handler/image-20240801091027552.png)

![image-20240801091049448](./.16_interrupt_handler/image-20240801091049448.png)

![image-20240801091116593](./.16_interrupt_handler/image-20240801091116593.png)

![image-20250123134110751](./.16_interrupt_handler/image-20250123134110751.png)

![image-20240801091149471](./.16_interrupt_handler/image-20240801091149471.png)



## irq_request_resources

![image-20240805182200707](./.16_interrupt_handler/image-20240805182200707.png)



# request_irq

![image-20240810122306310](./.16_interrupt_handler/image-20240810122306310.png)

## request_threaded_irq

![image-20240810122608715](./.16_interrupt_handler/image-20240810122608715.png)

![image-20240810122656898](./.16_interrupt_handler/image-20240810122656898.png)

![image-20240810122814372](./.16_interrupt_handler/image-20240810122814372.png)



# of irq



## of_irq_parse_one

![image-20240806151806263](./.16_interrupt_handler/image-20240806151806263.png)

![image-20240806151828600](./.16_interrupt_handler/image-20240806151828600.png)

new style

![image-20240806152147328](./.16_interrupt_handler/image-20240806152147328.png)

![image-20240806152239633](./.16_interrupt_handler/image-20240806152239633.png)

### of_irq_parse_raw

![image-20240806152421039](./.16_interrupt_handler/image-20240806152421039.png)

![image-20240806152443909](./.16_interrupt_handler/image-20240806152443909.png)

![image-20240806152519302](./.16_interrupt_handler/image-20240806152519302.png)

![image-20240806154325309](./.16_interrupt_handler/image-20240806154325309.png)

![image-20240806152600442](./.16_interrupt_handler/image-20240806152600442.png)

![image-20240806152635983](./.16_interrupt_handler/image-20240806152635983.png)

![image-20240806152653382](./.16_interrupt_handler/image-20240806152653382.png)



## irq_of_parse_and_map

![image-20240809151953047](./.16_interrupt_handler/image-20240809151953047.png)



### irq_create_of_mapping

![image-20240806154425359](./.16_interrupt_handler/image-20240806154425359.png)

#### of_phandle_args_to_fwspec

![image-20240806154517571](./.16_interrupt_handler/image-20240806154517571.png)

### irq_create_fwspec_mapping

![image-20240809145256098](./.16_interrupt_handler/image-20240809145256098.png)

![image-20240806154701052](./.16_interrupt_handler/image-20240806154701052.png)

![image-20240809143202783](./.16_interrupt_handler/image-20240809143202783.png)



#### irq_find_matching_fwspec

![image-20240807105916792](./.16_interrupt_handler/image-20240807105916792.png)

#### irq_domain_translate

![image-20240809143835321](./.16_interrupt_handler/image-20240809143835321.png)



## struct



### of_phandle_args

![image-20240806152815955](./.16_interrupt_handler/image-20240806152815955.png)



#### irq_fwspec

![image-20240809144414372](./.16_interrupt_handler/image-20240809144414372.png)



#### device_node

![image-20240806154907752](./.16_interrupt_handler/image-20240806154907752.png)

#### fwnode_handle

![image-20240806154957806](./.16_interrupt_handler/image-20240806154957806.png)

##### fwnode_operations

![image-20240806155051400](./.16_interrupt_handler/image-20240806155051400.png)

![image-20240806155138783](./.16_interrupt_handler/image-20240806155138783.png)

![image-20240806155155972](./.16_interrupt_handler/image-20240806155155972.png)

![image-20240806155214039](./.16_interrupt_handler/image-20240806155214039.png)





### irq_desc

![image-20240805182335724](./.16_interrupt_handler/image-20240805182335724.png)

![image-20240805182359897](./.16_interrupt_handler/image-20240805182359897.png)

![image-20240805182432234](./.16_interrupt_handler/image-20240805182432234.png)



#### irq_data

![image-20240805182456952](./.16_interrupt_handler/image-20240805182456952.png)

##### irq_chip

![image-20240805182901545](./.16_interrupt_handler/image-20240805182901545.png)

![image-20240805183038829](./.16_interrupt_handler/image-20240805183038829.png)

![image-20240805183110658](./.16_interrupt_handler/image-20240805183110658.png)

![image-20240805183130119](./.16_interrupt_handler/image-20240805183130119.png)



##### irq_domain

The `irq_domain` is responsible for reverse mapping of the `hwirq` into the Linux IRQ number space.

![image-20240809102327465](./.16_interrupt_handler/image-20240809102327465.png)

![image-20240809102159036](./.16_interrupt_handler/image-20240809102159036.png)

##### irq_domain_ops

![image-20240808111347758](./.16_interrupt_handler/image-20240808111347758.png)

![image-20240808111450936](./.16_interrupt_handler/image-20240808111450936.png)

#### irqaction

![image-20240805182549169](./.16_interrupt_handler/image-20240805182549169.png)



# irq domain

A mechanism to separate controller-local interrupt numbers, called hardware irq's, from Linux IRQ numbers. The `irq_domain` object is responsible for `hwirq` to `irq` translation.

![image-20240807111321819](./.16_interrupt_handler/image-20240807111321819.png)

![image-20240808190118351](./.16_interrupt_handler/image-20240808190118351.png)

## irq_domian Hierarchy

Hierarchy IRQ domain:

On some architectures, there may be multiple interrupt controllers involved in delivering an interrupt from the device to the target CPU. Here is a typical interrupt delivering path on RISC-V platforms:

​	UART device --> PLIC -> riscv-intc -> hart

There are two interrupt controllers involved: 

1. PLIC (Platform level interrupt controller)
2. riscv-intc (hart interrupt controller)

To support such a hardware topology and make software architecture match hardware architecture, an `irq_domain` data structure is built for each interrupt controller and those `irq_domain`s are organized into hierarchy. When building `irq_domain` hierarchy, the `irq_domain` near to the device is child and the `irq_domain` near to CPU is parent. So a hierarchy structure as below will be built for the example above:

```
	CPU Vector `irq_domain` (root `irq_domain` to manage CPU vectors)
			^
			|
	PLIC `irq_domain` (manage external interrupts)
```

![image-20240808111800866](./.16_interrupt_handler/image-20240808111800866.png)


There are four major interfaces to use hierarchy irq_domain:

1) `irq_domain_alloc_irqs()`: allocate IRQ **descriptors** and interrupt
   controller related resources to deliver these interrupts.
2) `irq_domain_free_irqs()`: free IRQ descriptors and interrupt controller
   related resources associated with these interrupts.
3) `irq_domain_activate_irq()`: activate interrupt controller hardware to
   deliver the interrupt.
4) `irq_domain_deactivate_irq()`: deactivate interrupt controller hardware
   to stop delivering the interrupt.

With support of hierarchy `irq_domian` and hierarchy `irq_data` ready, **an `irq_domain` structure is built for each interrupt controller**, and an `irq_data` structure is allocated for each `irq_domain` associate with an IRQ. Now we could go one step further to support stacked(hierarchy) `irq_chip`. That is, an `irq_chip` is associated with each `irq_data` along the hierarchy. A child `irq_chip` may implement a required action by itself or by cooperating with its parent `irq_chip`.

With stacked `irq_chip`, interrupt controller driver only needs to deal with the hardware managed by itself and may ask for services from its parent `irq_chip` when needed. So we could achieve a much cleaner software architecture.



### irq_domain_alloc_irqs

![image-20240808112149845](./.16_interrupt_handler/image-20240808112149845.png)

### __irq_domain_alloc_irqs

![image-20240809092028256](./.16_interrupt_handler/image-20240809092028256.png)

![image-20240809092121471](./.16_interrupt_handler/image-20240809092121471.png)

#### irq_domain_alloc_descs

![image-20240809093300604](./.16_interrupt_handler/image-20240809093300604.png)

#### __irq_alloc_descs

![image-20240809092856901](./.16_interrupt_handler/image-20240809092856901.png)

![image-20240809093049212](./.16_interrupt_handler/image-20240809093049212.png)

![image-20240808191536768](./.16_interrupt_handler/image-20240808191536768.png)

![image-20240808191603627](./.16_interrupt_handler/image-20240808191603627.png)

##### alloc_descs

![image-20240809094604213](./.16_interrupt_handler/image-20240809094604213.png)

##### alloc_desc

![image-20240809094708147](./.16_interrupt_handler/image-20240809094708147.png)

##### irq_insert_desc

In this way, IRQ numbers are connected to its corresponding `irq_desc`.

![image-20240809094739806](./.16_interrupt_handler/image-20240809094739806.png)

#### irq_domain_alloc_irq_data

![image-20240808191808555](./.16_interrupt_handler/image-20240808191808555.png)

#### irq_domain_alloc_irqs_hierarchy

![image-20240808191900621](./.16_interrupt_handler/image-20240808191900621.png)

#### irq_domain_trim_hierarchy

![image-20240808191948340](./.16_interrupt_handler/image-20240808191948340.png)

#### irq_domain_insert_irq

![image-20240808192047822](./.16_interrupt_handler/image-20240808192047822.png)

##### irq_domain_set_mapping

![image-20240808192205885](./.16_interrupt_handler/image-20240808192205885.png)



## irq_domain_add/create_*

![image-20240807112520065](./.16_interrupt_handler/image-20240807112520065.png)

![image-20240807112626617](./.16_interrupt_handler/image-20240807112626617.png)

### irq_domain_add_linear

![image-20240807112656295](./.16_interrupt_handler/image-20240807112656295.png)

![image-20240807140806334](./.16_interrupt_handler/image-20240807140806334.png)

![image-20240807140844769](./.16_interrupt_handler/image-20240807140844769.png)

![image-20240807140900480](./.16_interrupt_handler/image-20240807140900480.png)

![image-20240807140914176](./.16_interrupt_handler/image-20240807140914176.png)

![image-20240807140933672](./.16_interrupt_handler/image-20240807140933672.png)



## __irq_domain_add

![image-20240807111954764](./.16_interrupt_handler/image-20240807111954764.png)

![image-20240807112020137](./.16_interrupt_handler/image-20240807112020137.png)

![image-20240807112138213](./.16_interrupt_handler/image-20240807112138213.png)



### irq_create_mapping

In most cases, the `irq_domain` will begin with empty without any mappings between hwirq and IRQ numbers. Mappings are added to the `irq_domain` by calling `irq_create_mapping()`.

![image-20240809122633167](./.16_interrupt_handler/image-20240809122633167.png)

#### irq_create_mapping_affinity

![image-20240809122704319](./.16_interrupt_handler/image-20240809122704319.png)

![image-20240809122724411](./.16_interrupt_handler/image-20240809122724411.png)

#### irq_domain_associate

![image-20240809141057733](./.16_interrupt_handler/image-20240809141057733.png)

![image-20240809141150084](./.16_interrupt_handler/image-20240809141150084.png)



## irq_desc_tree

![image-20240808094726322](./.16_interrupt_handler/image-20240808094726322.png)



# struct irq_desc

interrupt descriptor

![image-20250123101007696](./.16_interrupt_handler/image-20250123101007696.png)

![image-20250123101111344](./.16_interrupt_handler/image-20250123101111344.png)

![image-20250123101141522](./.16_interrupt_handler/image-20250123101141522.png)



## irq_desc_get_chip

![image-20250123101451226](./.16_interrupt_handler/image-20250123101451226.png)



## struct irq_data

![image-20250123101537959](./.16_interrupt_handler/image-20250123101537959.png)



### irq_chip

![image-20250123101624770](./.16_interrupt_handler/image-20250123101624770.png)

![image-20250123101703971](./.16_interrupt_handler/image-20250123101703971.png)

![image-20250123101725239](./.16_interrupt_handler/image-20250123101725239.png)

![image-20250123101743023](./.16_interrupt_handler/image-20250123101743023.png)



# riscv_intc



## riscv_intc_init

![image-20240809100613161](./.16_interrupt_handler/image-20240809100613161.png)

### irq_domain_add_linear

Allocate and register a linear revmap `irq_domain`.

See `irq_domain_add/create_*` section for more details.

#### riscv_intc_domain_ops

![image-20240809145125001](./.16_interrupt_handler/image-20240809145125001.png)



### set_handle_irq

![image-20240809103251555](./.16_interrupt_handler/image-20240809103251555.png)

![image-20240809103314516](./.16_interrupt_handler/image-20240809103314516.png)

![image-20240809103407266](./.16_interrupt_handler/image-20240809103407266.png)



## riscv_intc_irq

The `cause` is holding the `hwirq` value here.

![image-20240809095750822](./.16_interrupt_handler/image-20240809095750822.png)

## handle_domain_irq

![image-20240809101002706](./.16_interrupt_handler/image-20240809101002706.png)

![image-20240809101045346](./.16_interrupt_handler/image-20240809101045346.png)

## __handle_domain_irq

![image-20250123102246643](./.16_interrupt_handler/image-20250123102246643.png)



### set_irq_regs

![image-20240809102725070](./.16_interrupt_handler/image-20240809102725070.png)

![image-20240809104100023](./.16_interrupt_handler/image-20240809104100023.png)



### irq_find_mapping

When an interrupt is received, `irq_find_mapping()` function should be used to find the Linux IRQ number from the hwirq number.

![image-20240807173134024](./.16_interrupt_handler/image-20240807173134024.png)





# PLIC



## plic_init

![image-20240809152611360](./.16_interrupt_handler/image-20240809152611360.png)

![image-20240809153442919](./.16_interrupt_handler/image-20240809153442919.png)

![image-20240809152717196](./.16_interrupt_handler/image-20240809152717196.png)

![image-20240809152739109](./.16_interrupt_handler/image-20240809152739109.png)



### irq_set_chained_handler

![image-20240809153125181](./.16_interrupt_handler/image-20240809153125181.png)

### __irq_set_handler

![image-20240809153234612](./.16_interrupt_handler/image-20240809153234612.png)

### __irq_do_set_handler

![image-20240809153347725](./.16_interrupt_handler/image-20240809153347725.png)

![image-20240809153559767](./.16_interrupt_handler/image-20240809153559767.png)



## plic_handle_irq

![image-20250123103415801](./.16_interrupt_handler/image-20250123103415801.png)





# generic_handle_irq

This function applies to all levels of irq chip.

![image-20240807173306329](./.16_interrupt_handler/image-20240807173306329.png)



## irq_to_desc

irq-desc pairs are stored in a **radix tree** structure.

![image-20250123105104935](./.16_interrupt_handler/image-20250123105104935.png)



## generic_handle_irq_desc

![image-20240807173358802](./.16_interrupt_handler/image-20240807173358802.png)

.handle_irq

![image-20250123104131416](./.16_interrupt_handler/image-20250123104131416.png)
