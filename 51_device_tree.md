# struct device

![image-20241202145246053](./.51_device_tree/image-20241202145246053.png)

![image-20241202145412535](./.51_device_tree/image-20241202145412535.png)

![image-20241203095407673](./.51_device_tree/image-20241203095407673.png)

![image-20241202145633966](./.51_device_tree/image-20241202145633966.png)

![image-20241202145703863](./.51_device_tree/image-20241202145703863.png)

## device_private

![image-20241203095932588](./.51_device_tree/image-20241203095932588.png)



## platform_device

![image-20241203104145837](./.51_device_tree/image-20241203104145837.png)

### resource

![image-20241203105019873](./.51_device_tree/image-20241203105019873.png)



# struct device_node

The `.of_node` member in `struct device`

![image-20241203105811528](./.51_device_tree/image-20241203105811528.png)



## of_node_get

![image-20241203105730619](./.51_device_tree/image-20241203105730619.png)

![image-20241203110204885](./.51_device_tree/image-20241203110204885.png)



## of_find_node_by_path

![image-20241203111458772](./.51_device_tree/image-20241203111458772.png)

### of_find_node_opts_by_path

![image-20241203111800485](./.51_device_tree/image-20241203111800485.png)

![image-20241203111942153](./.51_device_tree/image-20241203111942153.png)

### of_root

![image-20241203112546702](./.51_device_tree/image-20241203112546702.png)



### __of_find_node_by_full_path

![image-20241203112307835](./.51_device_tree/image-20241203112307835.png)

### __of_find_node_by_path

![image-20241203112514571](./.51_device_tree/image-20241203112514571.png)



## of_get_next_available_child

![image-20241203135558341](./.51_device_tree/image-20241203135558341.png)



## of_address_to_resource

![image-20241203101442372](./.51_device_tree/image-20241203101442372.png)

### of_get_address

![image-20241203101732491](./.51_device_tree/image-20241203101732491.png)

### __of_address_to_resource

![image-20241203102323976](./.51_device_tree/image-20241203102323976.png)



## of_irq_to_resource_tables

![image-20241203104321714](./.51_device_tree/image-20241203104321714.png)

### of_irq_to_resource

![image-20241203104518125](./.51_device_tree/image-20241203104518125.png)

### of_irq_get

![image-20241203104842142](./.51_device_tree/image-20241203104842142.png)



## of_property_read_bool

![image-20241204103811403](./.51_device_tree/image-20241204103811403.png)



![image-20241204103938708](./.51_device_tree/image-20241204103938708.png)



![image-20241204104012880](./.51_device_tree/image-20241204104012880.png)

# of_find_node_by_phandle

![image-20241210090706371](./.51_device_tree/image-20241210090706371.png)

## of_phandle_cache_hash

![image-20241210091042177](./.51_device_tree/image-20241210091042177.png)



# of_alias_scan

![image-20241206174047115](./.51_device_tree/image-20241206174047115.png)

![image-20241212133952099](./.51_device_tree/image-20241212133952099.png)

In device tree `aliases` node, the alias names are comprised of stem name and id.

## of_alias_add

![image-20241206172130270](./.51_device_tree/image-20241206172130270.png)



# of_property_read_u32

![image-20241209091119433](./.51_device_tree/image-20241209091119433.png)

## of_property_read_u32_array

![image-20241209091234731](./.51_device_tree/image-20241209091234731.png)

## of_property_read_variable_u32_array

![image-20241209093238778](./.51_device_tree/image-20241209093238778.png)

If there are errors in `of_find_property_value_of_size`, the `out_values` will be left untouched, which is presumablly having a default value of zero.

## of_find_property_value_of_size

![image-20241209091747147](./.51_device_tree/image-20241209091747147.png)

### -EINVAL

If the pointer is NULL, `ERR_PTR(-22)` will be returned

![image-20241209092459575](./.51_device_tree/image-20241209092459575.png)

### ERR_PTR

![image-20241209092955923](./.51_device_tree/image-20241209092955923.png)



# of_get_property

![image-20241211094833358](./.51_device_tree/image-20241211094833358.png)

# of_find_property

![image-20241209091838370](./.51_device_tree/image-20241209091838370.png)

## __of_find_property

![image-20241209091903407](./.51_device_tree/image-20241209091903407.png)



## of_prop_cmp

![image-20250412231305571](./.51_device_tree/image-20250412231305571.png)



# of_parse_phandle*

## of_parse_phandle

![image-20241204113039445](./.51_device_tree/image-20241204113039445.png)

## of_parse_phandle_with_args

![image-20241204145905444](./.51_device_tree/image-20241204145905444.png)

![image-20241204145923165](./.51_device_tree/image-20241204145923165.png)

## of_parse_phandle_with_args_map

![image-20241204150031716](./.51_device_tree/image-20241204150031716.png)

![image-20241210091833009](./.51_device_tree/image-20241210091833009.png)

![image-20241206100105199](./.51_device_tree/image-20241206100105199.png)

![image-20241206100124821](./.51_device_tree/image-20241206100124821.png)

![image-20241206100140839](./.51_device_tree/image-20241206100140839.png)

`stem_name` = "gpio" makes `cells_name` = "#gpio-cells", which is set to `2` in device tree.



## of_parse_phandle_with_fixed_args

![image-20241204150135035](./.51_device_tree/image-20241204150135035.png)



### __of_parse_phandle_with_args

![image-20241204112915956](./.51_device_tree/image-20241204112915956.png)

![image-20241204112935813](./.51_device_tree/image-20241204112935813.png)

### of_for_each_phandle

![image-20241204112743047](./.51_device_tree/image-20241204112743047.png)

#### of_phandle_iterator_init

![image-20241206100415191](./.51_device_tree/image-20241206100415191.png)



#### of_phandle_iterator_next

![image-20241204112547640](./.51_device_tree/image-20241204112547640.png)

![image-20241204112615954](./.51_device_tree/image-20241204112615954.png)

![image-20241204112637923](./.51_device_tree/image-20241204112637923.png)



### of_phandle_iterator_args

![image-20241210091703570](./.51_device_tree/image-20241210091703570.png)

## struct of_phandle_iterator

![image-20241211095155535](./.51_device_tree/image-20241211095155535.png)

### phandle

It's actually a pointer to nodes.

![image-20241211095231157](./.51_device_tree/image-20241211095231157.png)



# gpiod_get_array

![image-20241204143733599](./.51_device_tree/image-20241204143733599.png)

![image-20241204143503216](./.51_device_tree/image-20241204143503216.png)



![image-20241204143057379](./.51_device_tree/image-20241204143057379.png)



# of_gpio_count

![image-20241204142818870](./.51_device_tree/image-20241204142818870.png)

## of_gpio_named_count

![image-20241204142326898](./.51_device_tree/image-20241204142326898.png)



## of_count_phandle_with_args

![image-20241204133903377](./.51_device_tree/image-20241204133903377.png)

![image-20241204133920498](./.51_device_tree/image-20241204133920498.png)

****

# of_core_init

![image-20241203112639589](./.51_device_tree/image-20241203112639589.png)

## unflatten_device_tree

![image-20241203112857921](./.51_device_tree/image-20241203112857921.png)

## __unflatten_device_tree

![image-20241203113037955](./.51_device_tree/image-20241203113037955.png)

![image-20241203113104448](./.51_device_tree/image-20241203113104448.png)

### unflatten_dt_nodes

![image-20241203113922170](./.51_device_tree/image-20241203113922170.png)

![image-20241203114012931](./.51_device_tree/image-20241203114012931.png)

### populate_node

![image-20241205090310853](./.51_device_tree/image-20241205090310853.png)



#### unflatten_dt_alloc

![image-20241203114148195](./.51_device_tree/image-20241203114148195.png)



#### of_node_init

![image-20241203134638117](./.51_device_tree/image-20241203134638117.png)



#### populate_properties

![image-20241203114210976](./.51_device_tree/image-20241203114210976.png)

![image-20241203114234325](./.51_device_tree/image-20241203114234325.png)

![image-20241203114304549](./.51_device_tree/image-20241203114304549.png)



# struct fwnode_handle

The `.fwnode` member in `struct device`

![image-20241202144423591](./.51_device_tree/image-20241202144423591.png)

## fwnode_operations

![image-20241202144513457](./.51_device_tree/image-20241202144513457.png)

![image-20241202144607709](./.51_device_tree/image-20241202144607709.png)

![image-20241202144633208](./.51_device_tree/image-20241202144633208.png)



# of_fwnode_ops

![image-20241203134757464](./.51_device_tree/image-20241203134757464.png)

![image-20241204171149682](./.51_device_tree/image-20241204171149682.png)



## .get_next_child_node

![image-20241203135313463](./.51_device_tree/image-20241203135313463.png)



## .get_named_child_node

![image-20241203135038343](./.51_device_tree/image-20241203135038343.png)



## .get_reference_args

![image-20241204151018831](./.51_device_tree/image-20241204151018831.png)

### fwnode_property_get_reference_args

![image-20241204151134746](./.51_device_tree/image-20241204151134746.png)

### fwnode_find_reference

![image-20241204151242071](./.51_device_tree/image-20241204151242071.png)

### fwnode_devcon_match

![image-20241204151321745](./.51_device_tree/image-20241204151321745.png)

### fwnode_connection_find_match

![image-20241204151356639](./.51_device_tree/image-20241204151356639.png)

### device_connection_find_match

![image-20241204151435562](./.51_device_tree/image-20241204151435562.png)



## .device_get_match_data

![image-20241204171231749](./.51_device_tree/image-20241204171231749.png)

### of_device_get_match_data

![image-20241204171300126](./.51_device_tree/image-20241204171300126.png)

![image-20241204171326173](./.51_device_tree/image-20241204171326173.png)

![image-20241204171347517](./.51_device_tree/image-20241204171347517.png)



## .add_links

### of_fwnode_add_links

![image-20241211152017735](./.51_device_tree/image-20241211152017735.png)



### of_link_to_suppliers

![image-20241211151840070](./.51_device_tree/image-20241211151840070.png)





### of_link_property

![image-20241211151400967](./.51_device_tree/image-20241211151400967.png)

![image-20241211151421230](./.51_device_tree/image-20241211151421230.png)



![image-20241211150652315](./.51_device_tree/image-20241211150652315.png)

![image-20241211150732147](./.51_device_tree/image-20241211150732147.png)

expand to:

parse_gpio()

parse_gpios()

![image-20241211151230112](./.51_device_tree/image-20241211151230112.png)

### parse_suffix_prop_cells

![image-20241211145342986](./.51_device_tree/image-20241211145342986.png)

#### of_parse_phandle_with_args

See more details in `of_parse_phandle*` section.

![image-20241204145905444](./.51_device_tree/image-20241204145905444.png)

![image-20241204145923165](./.51_device_tree/image-20241204145923165.png)
