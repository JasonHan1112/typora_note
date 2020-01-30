2019/9/6
# raspberry jtag的使用
- 登陆
ssh root@172.18.26.30
passwd:higon123;
- 启动脚本
cd /home/higon/HigonDFXToolSuite
./ft4232hl_run.sh

- 运行报错
vim **dhyana_board_config.tcl** change die num and socket and verson

- 启动input进程
  - **another ternminal**
telnet localhost 4444
jtag_init
select_tap 1
select_die0
- input command
mp0_read_via_smnif 0xaddrxxx
ensmn
rdsmn 0xaddrxxx
rwsmn 0xaddrxxx

# raspberry_jtag shell example
```
#!/bin/tclsh

#until val == [addr]
proc reset_phy_lane {} {
        wrsmn 0x12009864 0
        wrsmn 0x12029864 0
        wrsmn 0x12049864 0
        wrsmn 0x12069864 0
        wrsmn 0x12089864 0
        wrsmn 0x12109864 0
        wrsmn 0x12129864 0
        wrsmn 0x12149864 0
        wrsmn 0x12169864 0
        wrsmn 0x12189864 0

}
proc send_dxio_message { p0 p1 p2 p3 p4 p5 } {
        #clear response
        wrsmn 0x3b10564 0
        #fill parameters
        wrsmn 0x3b10598 $p0
        wrsmn 0x3b1059c $p1
        wrsmn 0x3b105a0 $p2
        wrsmn 0x3b105a4 $p3        
        wrsmn 0x3b105a8 $p4        
        wrsmn 0x3b105ac $p5
        #fill msg_id, dxio=0xf
        wrsmn 0x3b10528 0xf

}
proc read_dxio_message {} {
        set value [rdsmn 0x3b10564]
        echo "response: $value"
        set value [rdsmn 0x3b10598]
        echo "p0: $value"
        set value [rdsmn 0x3b1059c]
        echo "p1: $value"
        set value [rdsmn 0x3b105a0]
        echo "p2: $value"
        set value [rdsmn 0x3b105a4]
        echo "p3: $value"
        set value [rdsmn 0x3b105a8]
        echo "p4: $value"
        set value [rdsmn 0x3b105ac]
        echo "p5: $value"

}
proc wait_until_set { val addr } {
        set tmp [rdsmn $addr]
        echo "in wait tmp = $tmp"
        while {$tmp != $val} {
                echo "in wait circle"
                set tmp [rdsmn $addr]
                echo "addr = $addr, val = $tmp"
                echo "***************waiting value**************"
                read_dxio_message
                echo "###############waiting value###############"
                sleep 1000    
                echo "waiting..."
        }
}

proc dxio_seq {} {
        
        #reset phy_lane
        reset_phy_lane
        
        #1
        send_dxio_message 0x0 0x0 0x0 0x0 0x0 0x0
        read_dxio_message
        wait_until_set 1 0x3b10564
        #2
        send_dxio_message 0x13 0x0 0x0 0x0 0x0 0x0
        read_dxio_message
        wait_until_set 1 0x3b10564
        #add 1
        send_dxio_message 0x22 0xE 0x1 0x0 0x0 0x0
        read_dxio_message
        wait_until_set 1 0x3b10564
        #add 2
        send_dxio_message 0x22 0xF 0x1 0x0 0x0 0x0
        read_dxio_message
        wait_until_set 1 0x3b10564
        #3
        send_dxio_message 0x22 0x10 0x1 0x0 0x0 0x0
        read_dxio_message
        wait_until_set 1 0x3b10564
        #4 
        send_dxio_message 0x24 0x0 0x0 0x0 0x0 0x0
        read_dxio_message
        wait_until_set 1 0x3b10564
        #5
        send_dxio_message 0x23 0x0 0x0 0x0 0x1 0x2
        read_dxio_message
        wait_until_set 1 0x3b10564
        #6
        send_dxio_message 0x23 0x0 0x0 0x0 0x1 0x5
        read_dxio_message
        wait_until_set 1 0x3b10564
        #7
        send_dxio_message 0x23 0x0 0x0 0x0 0x1 0x3
        read_dxio_message
        wait_until_set 1 0x3b10564
        #8
        send_dxio_message 0x307 0x0 0x0 0x0 0x0 0x0
        read_dxio_message
        wait_until_set 1 0x3b10564
        #9
        send_dxio_message 0x9 0x0 0x0 0x0 0x0 0x0
        read_dxio_message
        wait_until_set 1 0x3b10564

        while {1} {
                echo "in a circle"

                #10
                send_dxio_message 0x308 0x0 0x0 0x0 0x0 0x0
                read_dxio_message
                wait_until_set 1 0x3b10564
                #11
                send_dxio_message 0x9 0x0 0x0 0x0 0x0 0x0
                read_dxio_message
                wait_until_set 1 0x3b10564

                set tmp_val [rdsmn 0x3b105a0]

                set tmp_val1 [rdsmn 0x3b1059c]
                if {$tmp_val1 == 0x102} {

                        if {$tmp_val == 0xd} {
                        echo " 0xd"
                        break
                        }
                }
                sleep 1000
                
        }

}
```


