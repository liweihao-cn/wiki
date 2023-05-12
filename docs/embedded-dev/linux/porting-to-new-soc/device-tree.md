# FDT

<a id="device-tree"></id>

<a id="flow_of_kernel_process_dt"></a>

内核对设备树的处理的主要流程

```c
start_kernel
  -->setup_arch
    -->setup_machine_fdt                         /* arch/arm/kernel/devtree.c */
      -->early_init_dt_verify
      -->of_flat_dt_match_machine
      -->early_init_dt_scan_nodes
        -->of_scan_flat_dt(early_init_dt_scan_chosen, boot_command_line);
           /* Retrieve various information from the /chosen node */
        -->of_scan_flat_dt(early_init_dt_scan_root, NULL);
           /* Initialize {size,address}-cells info */
        -->of_scan_flat_dt(early_init_dt_scan_memory, NULL);
           /* Setup memory, calling early_init_dt_add_memory_arch */
    -->unflatten_device_tree                              /* drivers/of/fdt.c */
      -->__unflatten_device_tree
        -->unflatten_dt_nodes(blob, NULL, dad, NULL);
           /* First pass, scan for size */
        -->unflatten_dt_nodes(blob, mem, dad, mynodes);
           /* Second pass, do actual unflattening */
  -->rest_init
    -->kernel_thread(kernel_init, NULL, CLONE_FS);
      -->kernel_init
        -->kernel_init_freeable
          -->do_basic_setup
            -->do_initcalls
              -->do_initcall_level
                -->of_platform_default_populate_init /* drivers/of/platform.c */
                  -->of_platform_default_populate
                    -->of_platform_populate
                      -->of_platform_bus_create
                        -->...
```

第一部分完成设备树的扁平化处理；第二部分扫描整个设备树，并为其中符合要求的节点生成平台设备。

第二部分中符合要求的节点包括根节点下有compatible属性的节点以及compatible属性匹配下面结构体中属性的节点。

```c
const struct of_device_id of_default_bus_match_table[] = {
	{ .compatible = "simple-bus", },
	{ .compatible = "simple-mfd", },
	{ .compatible = "isa", },
#ifdef CONFIG_ARM_AMBA
	{ .compatible = "arm,amba-bus", },
#endif /* CONFIG_ARM_AMBA */
	{} /* Empty terminated list */
};

```

