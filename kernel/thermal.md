# thermal frame
- 获取温度的设备：在 Thermal 框架中被抽象为 Thermal Zone Device;
- 控制温度的设备：在 Thermal 框架中被抽象为 Thermal Cooling Device;
- 控制温度策略：在 Thermal 框架中被抽象为 Thermal Governor;
- Thermal Core 就负责把这些整合在一起。
## 设备注册thermal的简单流程
１．注册函数 ,thermal Core 通过对外提供注册的接口，让 thermal zone device、thermal cooling device、thermal governor 注册进来。
```c
struct thermal_zone_device *thermal_zone_device_register(const char *, int, int,
                void *, struct thermal_zone_device_ops *,
                const struct thermal_zone_params *, int, int);

struct thermal_cooling_device *thermal_cooling_device_register(char *, void *,
                const struct thermal_cooling_device_ops *);

int thermal_register_governor(struct thermal_governor *);
```

２．Thermal zone/cooling device 注册的过程中 thermal core 会调用绑定函数，绑定的过程最主要是一个 cooling device 绑定到一个 thremal_zone 的触发点上
```c
int thermal_zone_bind_cooling_device(struct thermal_zone_device *tz)
{
        struct thermal_instance *dev;
        struct thermal_instance *pos;
        struct thermal_zone_device *pos1;
        struct thermal_cooling_device *pos2;
        unsigned long max_state;
        int result;

	...

	/* thermal_instace 就是绑定之后的实例 */
        dev =
            kzalloc(sizeof(struct thermal_instance), GFP_KERNEL);
        if (!dev)
                return -ENOMEM;
        dev->tz = tz;
        dev->cdev = cdev;
        dev->trip = trip;
	    dev->upper = upper;

	...

            sysfs_create_link(&tz->device.kobj, &cdev->device.kobj, dev->name);
        if (result)
                goto release_idr;

        sprintf(dev->attr_name, "cdev%d_trip_point", dev->id);
        sysfs_attr_init(&dev->attr.attr);

	...

        list_for_each_entry(pos, &tz->thermal_instances, tz_node)
            if (pos->tz == tz && pos->trip == trip && pos->cdev == cdev) {
                result = -EEXIST;
                break;
        }
        if (!result) {
		/* 绑定完了就添加到链表中 */
                list_add_tail(&dev->tz_node, &tz->thermal_instances);
                list_add_tail(&dev->cdev_node, &cdev->thermal_instances);
        }

	...

        return result;
}
```
３．Thermal core 使能 delayed_work 循环处理 , 使整个 thermal 控制流程运转起来。

```c
static void thermal_zone_device_check(struct work_struct *wrok)
{
        struct thermal_zone_device *tz = container_of(work, struct
                                                      thermal_zone_device,
                                                      poll_queue.work);
	/* 处理函数 */
        thermal_zone_device_update(tz);
}

void thermal_zone_device_update(struct thermal_zone_device *tz)
{
        int count;
        if (!tz->ops->get_temp)
                return;
	/* 更新温度 */
        update_temperature(tz);-
        for (count = 0; count < tz->trips; count++)
		/* 处理触发点，这里面就会调到具体的 governor */
                handle_thermal_trip(tz, count);
}

static void thermal_zone_device_set_polling(struct thermal_zone_device *tz, int delay)
{
        if (delay > 1000)
		/* 更改 delayed_work 下次唤醒时间完成轮询 */
                mod_delayed_work(system_freezable_wq, &tz->poll_queue,
                                 round_jiffies(msecs_to_jiffies(delay)));
        else if (delay)
                mod_delayed_work(system_freezable_wq, &tz->poll_queue,
                                 msecs_to_jiffies(delay));
        else
                cancel_delayed_work(&tz->poll_queue);
}

```
## thermal initiallize
- thermal_init//__init系统初始化时执行
  |
  thermal_register_governors//向系统中注册支持的gonvernors
      |
      thermal_gov_step_wise_register//注册step_wise gonvernors
          |
          thermal_gov_power_allocator_register(&thermal_gov_step_wise)//"step_wise".throttle=step_wise_throttle，用法简介：thermal_zone_device_update->handle_thermal_trip->monitor_thermal_zone->governor->throttle执行相关函数->thermal_zone_device_set_polling->mod_delayed_work唤醒队列poll_queue，队列中的thermal_zone_device_check->thermal_zone_device_update->handle_thermal_trip->monitor_thermal_zone->再次polling，循环调用
      |
      thermal_gov_fair_share_register//注册fair_share gonvernors
          |
          thermal_register_governor(&thermal_gov_fair_share)//"fair_share", .throttle=fair_share_throttle,同上
      |
      thermal_gov_bang_bang_register//注册bang_bang gonvernors
          |
          thermal_register_governor(&thermal_gov_bang_bang);//"bang_bang", .throttle=bang_bang_control
      |
      thermal_gov_user_space_register//注册user_space gonvernors
          |
          thermal_register_governor(&thermal_gov_user_space);//"user_space", .throttle=notify_user_space//通过uevent通知用户空间
      |
      thermal_gov_power_allocator_register
          |
          thermal_register_governor(&thermal_gov_power_allocator);//"power_allocator", .throttle=power_allocator_throttle .bind_to_tz=power_allocator_bind
  |
  class_register//向系统中注册一个thermal_class
  |
  genetlink_init//注册一个通用netlink family
  |
  of_parse_thermal_zones//解析设备树中thermal-zones,
      |
      thermal_of_build_thermal_zone//根据设备树填充__thermal_zone. polling-delay-passive passive_delay polling-delay-fast polling_delay
          |
          of_get_child_by_name//获取trips ntrips
          |
          thermal_of_populate_trip//trip->"temperature" "hysteresis" 
              |
              thermal_of_get_trip_type//"type"
          |
          of_get_child_by_name//"cooling-maps"
          |
          thermal_of_populate_bind_params//"contribution" "cooling-device"...
      |
      of_property_read_u32(child, "sustainable-power", &prop)//"sustainable-power"
      |
      of_property_read_string(child, "governor", &name)//"governor"
      |
      thermal_zone_device_register//根据设备树的参数注册一个thermal_zone_device
          |
          tz->id = result; tz->ops = ops; tz->tzp = tzp; tz->device.class = &thermal_class; tz->devdata = devdata;
          tz->trips = trips; tz->passive_delay = passive_delay; tz->polling_delay = polling_delay;
          |
          thermal_zone_create_device_groups
          dev_set_name(&tz->device, "thermal_zone%d", tz->id); result = device_register(&tz->device);
          |
          result = thermal_set_governor(tz, governor);
          list_add_tail(&tz->node, &thermal_tz_list);
          INIT_DELAYED_WORK(&tz->poll_queue, thermal_zone_device_check);
          thermal_zone_device_reset(tz);
          thermal_zone_device_update(tz, THERMAL_EVENT_UNSPECIFIED);
  |
  register_pm_notifier//Add notifier to a blocking notifier chain

platform initial 
asr 利用platform设备来初始化thermal
name="stpu_thermal"
stpu_thermal_probe
|
res = platform_get_resource(pdev, IORESOURCE_MEM, 0);//获取temp sensor的硬件
info->tsen = devm_ioremap_resource(dev, res);//映射到内核中
of_property_read_u32(np, "emergent_reboot_threshold", &info->emergent_reboot_thrsh);//从设备树中获取info->emergent_reboot_thrsh
|
info->clkcore = devm_clk_get(dev, "thermal_core");//从设备树中获取thermal时钟
|
info->calib_offset = stpu_get_efuse_calib_offset();//从optee中获取thermal较准
|
ret = stpu_init_seggnsor(info);//将从设备树中获取的数据填入到temp sensor寄存器中，配置automode
|
info->irq = platform_get_irq_byname(pdev, "thermal_irq");//获取irp号
|
//向系统中注册中断回调stpu_thermal_irq,以及中断线程，中断回调(stpu_thermal_irq)首先执行负责读取温度更新阈值，清中断等操作return IRQ_WAKE_THREAD后，stpu_thermal_thread_irq执行，stpu_thermal_thread_irq->thermal_zone_device_update->handle_thermal_trip nocritical->throttle或者critical->thermal_emergency_poweroff
devm_request_threaded_irq(dev, info->irq, stpu_thermal_irq,stpu_thermal_thread_irq, IRQF_ONESHOT,dev_name(dev), info);
|
thermal_zone_device_register//根据sensor的数量(11)来注册thermal_zone_device_register
|
stpu_register_cpufreq_cdev//设备树中没有"cluster0-cooling-steps" "cluster1-cooling-steps" asr 没有注册
|
stpu_register_pu_cdev //设备树中没有"stpu0-cooling-steps" "stpu1-cooling-steps" asr 没有注册
|
stpu_register_fan_cdev//按照设备树中"fan-cooling-steps"注册设备
    |
    fan_register_cdev
        |
        of_fan_cooling_register
            |
            __fan_cooling_register
                |
                fan_dev->max_state = NUM_FAN_COOLING_STEPS - 1;
                fan_dev->cur_state = 0;
                fan_dev->limited_state = 0;
                |
                thermal_of_cooling_device_register(np, dev_name, fan_dev, &fan_cooling_ops);
                    |
                    __thermal_cooling_device_register//在其中注册一个thermal_cooling_device cdev 
                        |
                        cdev->ops = ops; cdev->device.class = &thermal_class; 
                        thermal_cooling_device_setup_sysfs(cdev);
                        dev_set_name(&cdev->device, "cooling_device%d", cdev->id);
                        result = device_register(&cdev->device);

        |
        thermal_zone_bind_cooling_device//bind a cooling device to a thermal zone
    |
    device_create_file//注册sysfs attribute
|
thermal_zone_device_update//更新温度，设置trip，handle trips
|
stpu_tsen_auto_mode//设置temp sensor auto mode
|
stpu_update_temp_thrshld//更新thresh hold stpu是用中断方式来进行trip的触发，因此要向相关寄存器中写入thrshld
thermal_zone_of_sensor_register
## 总结
asr通过thermal的中断来触发trip, 通过中断回调stpu_thermal_thread_irq->thermal_zone_device_update->handle_thermal_trip-> handle_critical_trips; handle_non_critical_trips; monitor_thermal_zone;
