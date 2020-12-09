# i2c简介
- introduction to i2c and SMBus
1. two-wire protocol (variable speed, up to 400kHz)
2. SMBus is based on the i2c protocol, a subset of i2c protocols and signaling 
- terminology
1. master chip: a node that starts communications with slaves. in kernel it is called adapter adapter drivers are in the drivers/i2c/busses/ subdirectory
2. algorithm: general code that implement a whole class of i2c adapters. each specific adapter driver either depends on an algorithm driver in the drivers/i2c/algos/
3. slave chip: a node that responds to communications when addressed by the master. in linux kernel called a client. client drivers are kept in a directory specific to the feature they provide, for example drivers/media/gpio/ for gpio expanders and drivers/media/i2c/ for video-related chips.
- simple send and receive transaction
S Addr Wr [A] Data [A] Data [A] ... [A] Data [A] P
S Addr Rd [A] [Data] [A] [Data] [A] ... [A] [Data] [A] NA P
- combined transactions
instead of a stop condition P a start condition S is sent and the transaction continue
S Addr Rd [A] NA S Addr Wr [A] Data [A] P
# the SMBus protocol
some adapters understand only the SMBus protocol, which is a subset from the i2c protocol. if you write a driver for some i2c device, please try to use the SMBus commands if at all possible, this makes it possible to use the device driver on both SMBus adapters and i2c adapters (the SMBus command set is automatically translated to i2c on i2c adapters, but plain i2c commands can not be handled at all on most pure SMBus adapters)
- SMBus quick command
I2C_FUNC_SMBUS_SUICK
S Addr Rd/Wr [A] P
- SMBus receive byte
i2c_smbus_read_byte() I2C_FUNC_SMBUS_READ_BYTE
S Addr Rd [A] [Data] NA P
- SMBus read word
i2c_smbus_read_word_data() I2C_FUNC_SMBUS_READ_WORD_DATA
from a designated register that is specified through the Comm byte.
S Addr Wr [A] Comm [A] S Addr Rd [A] [DataLow] A [DataHigh] NA P
- SMBus write byte
i2c_smbus_write_byte_data() I2C_FUNC_SMBUS_WRITE_BYTE_DATA
S Addr Wr [A] Comm [A] Data [A] P
- SMBus wirte word
i2c_smbus_write_word_data() I2C_FUNC_SMBUS_WRITE_WORD_DATA
S Addr Wr [A] Comm [A] DataLow [A] DataHigh [A] P
- SMBus process call
I2C_FUNC_SMBUS_PROC_CALL
S Addr Wr [A] Comm [A] DataLow [A] DataHigh [A]
                             S Addr Rd [A] [DataLow] A [DataHigh] NA P
- SMBus block read
i2c_smbus_read_block_data() I2C_FUNC_SMBUS_READ_BLOCK_DATA
S Addr Wr [A] Comm [A]
           S Addr Rd [A] [Count] A [Data] A [Data] A ... A [Data] NA P
- SMBus block wirte
i2c_smbus_write_block_data() I2C_FUNC_SMBUS_WRITE_BLOCK_DATA
S Addr Wr [A] Comm [A] Count [A] Data [A] ...
                             S Addr Rd [A] [Count] A [Data] ... A P
- ...
# how to instantiate i2c devices
- method 1: declare the i2c devices statically
1. via device tree
here, two devices are attached to the bus using a speed of 100kHz.
```c
i2c1: i2c@400a0000 {
        /* ... master properties skipped ... */
        clock-frequency = <100000>;

        flash@50 {
                compatible = "atmel,24c256";
                reg = <0x50>;
        };

        pca9532: gpio@60 {
                compatible = "nxp,pca9532";
                gpio-controller;
                #gpio-cells = <2>;
                reg = <0x60>;
        };
};
```
2. via acpi
3. in board files

- method 2: instantiate the devices explicitly
when a larger device uses an i2c bus for internal communication. a typical case is TV adapters. these can have a tuner, a video decoder, an audio decoder, etc.
this is done by filling a struct i2c_board_info and calling i2c_new_client_device()
```c
static struct i2c_board_info sfe4001_hwmon_info = {
      I2C_BOARD_INFO("max6647", 0x4e),
};

int sfe4001_init(struct efx_nic *efx)
{
      (...)
      efx->board_info.hwmon_client =
              i2c_new_client_device(&efx->i2c_adap, &sfe4001_hwmon_info);

      (...)
}
```
when you don't know for sure if an i2c device is present or not
i2c_new_scanned_device()
```c
static const unsigned short normal_i2c[] = { 0x2c, 0x2d, I2C_CLIENT_END };

static int usb_hcd_nxp_probe(struct platform_device *pdev)
{
      (...)
      struct i2c_adapter *i2c_adap;
      struct i2c_board_info i2c_info;

      (...)
      i2c_adap = i2c_get_adapter(2);
      memset(&i2c_info, 0, sizeof(struct i2c_board_info));
      strscpy(i2c_info.type, "isp1301_nxp", sizeof(i2c_info.type));
      isp1301_i2c_client = i2c_new_scanned_device(i2c_adap, &i2c_info,
                                                  normal_i2c, NULL);
      i2c_put_adapter(i2c_adap);
      (...)
}
```
method 4: instantiate from user-space
a sysfs interface was added to let the user provide the information: new_device and delete_deivce, both files are write only and you must write the right parameters to them in order to properly instantiate.
new_device: takes 2 prarmeters: the name of the i2c device (a string) and the address of the i2c device
delete_device: takes a single parameter: the address of the i2c devcie.
```c
# echo eeprom 0x50 > /sys/bus/i2c/devices/i2c-3/new_device
```
# implementing i2c device drivers
asr uses platform_bus to probe i2c adapters

# ASR i2c adapter initialization
1. platform device and platform driver
__init asr_i2c_init(void)初始化是执行，i2c的控制器在dtb中已经存在，platform_bus match后，asr_i2c_probe开始执行，填充asr_i2c_dev（其中有重要结构体i2c_adapter）,asr_i2c_parse_dt(解析设备树中的参数, asr_i2c->fast_mode, asr_i2c->high_mode, asr_i2c->apdcp, asr_i2c->dma_disable....), 填充asr_i2c->resrc(register map) asr_i2c->irq, asr_i2c->clk, asr_i2c->adapter, asr_i2c->dbgfs_mode. 然后调用i2c_add_numbered_adapter(其中会对adapter进行进一步的填充，以及对设备树以及i2c_bus中该i2c_adapter下的i2c_device进行进一步枚举)
2. 下边分析i2c_add_numbered_adapter
根据设备树中的asr,adapter-id来确定bus id，调用__i2c_add_numbered_adapter,进而调用i2c_register_adapter注册adapter,在i2c_regsiter_adapter中填充adap->bus=&i2c_bus_type, adap->dev.type=&i2c_adapter_type(建立sysfs有用)，of_i2c_register_device(adap)，注册设备树中的device，在其中读取设备树中的节点的配置信息，i2c_new_device创建并填充client，绑定client和adapter.i2c的设备创建完成。
3. 等待i2c driver注册到i2c总线后则会调用相关的i2c_driver中的probe完成初始化




















