# Physical Layer - Electrical
## back compatibility
  - Initial training is done at 2.5 GT/s for all devices.
  - Changing to other rates requires negotiation between the Link partners to determine the peak common frequency.
  - Root ports that support 8.0 GT/s are required to support both 2.5 and 5.0 GT/s as well. 
  - Downstream devices must obviously support 2.5 GT/s, but all higher rates are optional. This means that an 8 GT/s device is not required to support 5GT/s.
  - Refclk remains the same regardless of the data rate.
  - The  Transmitter  delivers  outbound  Symbols  on each Lane by converting the bit stream into two single‐ended electrical signals with opposite polarity. Receivers compare the two signals and, when the difference is sufficiently positive or negative, generate a one or zero internally to represent the intended serial bit stream to the rest of the Physical Layer.
  - Receivers sense this voltage as the input stream, but if it drops below a threshold value, it’s understood to represent the Electrical Idle Link  condition. 
## Clock Requirement
  -  both  Transmitter  and  Receiver  clocks  must  be  accurate  to within +/‐ 300 ppm (parts per million) of the center frequency. 
  -  Devices are allowed to derive their clocks from an external source, and the 100MHz Refclk is still optionally available for this purpose in the 3.0 spec. Using the refclk permits both link partners to readily maintain the 600ppm accuracy
  -  SSC is an optional technique used to modulate the clock frequency slowly over a  prescribed  range  to  spread  the  signal’s  EMI  (Electro‐Magnetic  Interference) across a range of frequencies rather than allowing it all to be concentrated at the center  frequency. 
  -  Common Refclk: *not be used ordinarily*
      - the  use  of  SSC  will  be  simplest  with  this  model  because maintaining the 600 ppm separation between the Tx and Rx clocks is easy if both follow the same modulated reference. 
      - the Refclk remains available during low‐power Link states L0s and L1 and that allows the Receiver’s CDR to maintain a semblance of the recovered clock even in the absence of a bit stream to supply  the  edges  in  the  data.  That,  in  turn,  keeps  the  local  PLLs from  drifting  as  much  as  they  otherwise  would,  resulting  in  a reduced recovery time back to L0 compared to the other clocking options.
![新建位图图像 (6).bmp](attachments\03249406.bmp)
  - Data Clocked Rx Architecture: This implementation  is  clearly  the  simplest  of  the  three  and  would  therefore ordinarily  be  preferred.  
![新建位图图像 (7).bmp](attachments\c3f9a5aa.bmp)
## Receive Detection
  - This step is a little unusual in the serial transport world because it’s easy  enough  to  send  packets  to  the  Link  partner  and  test  its  presence  by whether or not it responds. The motivation for this approach in PCIe, however, is to provide an automatic hardware assist in a test environment. 
  - If the proper load is detected, but the Link partner refuses to send TS1s and participate in Link Training, the component will assume that it must be in a test environment and will begin sending the Compliance Pattern to facilitate testing. 
  - Detection is accomplished by setting the Transmitter’s DC common mode voltage to one value and then changing it to another. Knowing the expected charge time when a Receiver is present, the logic compares the measured time against that. If a Receiver is attached, the charge time (RC time constant) is relatively long  due  to  the  Receiver’s  termination.  Otherwise,  the  charge  time  is  much shorter 
  - Detecting Receiver Presence
    - After reset or power‐up, Transmitters drive a stable voltage on the D+ and D‐ terminal.(DC common mode voltage)
    - Transmitters then change the common mode voltage in a positive direction by  no  more  than  the  VTX‐RCV‐DETECT  amount  of  600mV  specified  for  all three data rates.(change the DC common mode voltage)
    - Detect logic measures the charge time. (test the time)
 - DC Common Mode Voltage
    - after the Detect state of Link training the DC Common Mode Voltage must remain at the same voltage. The common mode voltage is turned off only in the L2 or L3 low‐power Link
states, in which main power to the device is removed. 
 - Differential voltage
   -  A logical one is indicated when the D+ signal is high and the D- signal low, while a logical zero is represented by driving the D+ signal low and the D‐ signal high.
   - • Logical 1 is signaled with a positive differential voltage. • Logical 0 is signaled with a negative differential voltage. 
   
   - The differential peak‐to‐peak voltage driven by the Transmitter VTX‐DIFFp‐p (see Table 13‐3 on page 489) is between 800 mV and 1200 mV (1300 mV for 8.0 GT/s). 
   - During Electrical Idle the Transmitter holds the differential peak voltage VTX‐IDLE‐DIFFp  (see  Table 13‐3  on  page 489)  very  near  zero  (0‐20  mV). 
   - The Receiver senses a logical one or zero, as well as Electrical Idle, by evaluating the voltage on the Link. The signal loss expected at high frequency means the Receiver must be able to sense an attenuated version of the signal
   - Differential Peak Voltage => VDIFFp = (max |VD+ ‐ VD‐ |); Differential Peak‐to‐Peak Voltage => VDIFFp‐p = 2 *(max |VD+ ‐ VD‐ |)

## How Does De-Emphasis Help?
  - De‐emphasis reduces the voltage for repeated bits in a bit stream. Although it sounds counter‐intuitive at first because this reduces the signal swing and thus the energy that reaches the Receiver, reducing the Transmitter voltage for these cases can substantially improve signal quality. 
  - The first bit of a series of same polarity bits is not de‐emphasized.
  - Only subsequent bits of the same polarity after the first bit are de‐emphasized.
![新建位图图像 (8).bmp](attachments\641e66b7.bmp)
  - solution for 2.5GT/s
    -  each  subsequent  bit  transmitted  after  the  first  bit  of  the  same polarity must be de‐emphasized by 3.5dB to accommodate this worst‐case loss budget.
    -  ![新建位图图像.bmp](attachments\0d268906.bmp)
    -  ![新建位图图像 (2).bmp](attachments\e9dec6b4.bmp)
  - solution for 5.0GT/s
    - ![新建位图图像 (3).bmp](attachments\5e5087fb.bmp) 
    - 3 mode for transmitter
      > - full swing mode with or without de-emphasis
      > - reduced swing mode with or without de-emphasis
        > ![新建位图图像 (4).bmp](attachments\1cf482c5.bmp)
  - solution for 8.0GT/s
    - Transmitter equalization becomes more complex and a handshake training procedure is used to adapt to the actual signaling environment rather than making assumptions about what will be needed.
    -  Basically, that process allows a Receiver to request that the Link partner’s Transmitter use a certain combination of coefficients and then the receiver tests how well the received signal looks and possibly proposes others if the result isn’t good enough.
  - 3 tap tx Equalizer Required  
![新建位图图像 (5).bmp](attachments\6a8fd6c6.bmp)
    
    - With this in mind, the three inputs can be described by their timing position as “pre‐cursor” for C‐1, “cursor” for C0, and “post‐cursor” for C+1, which combine to create an output based on the upcoming input, the current value, and the previous value. 
    - The effect of the coefficient values is to adjust the output voltage to create up to four different voltage levels to accommodate different signaling environments.
    - Va Vb Vc Vd
    > - When the bits on both sides of the cursor have the opposite polarity, the voltage will be Vd, the maximum voltage. 
    > - When a repeated string of bits is to be sent:
    > - The first bit will use Va, the next lower voltage to the maximum voltage Vd.
    > - Bits between the first and last bits use Vb, the lowest voltage.
    > - The last repeated bit before a polarity change uses Vc, the next higher voltage to the lowest voltage Vb.
    ![新建位图图像 (6).bmp](attachments\e5bf6eb0.bmp)
    
    - Preset and Ratios
  -  when the Link is preparing to change from a lower data rate to 8.0 GT/s, the Downstream Port sends EQ
TS2s that give the Upstream Port a set of preset values to use for its coefficients as a *starting point* from which to begin testing the Link signal quality. 
    ![新建位图图像 (7).bmp](attachments\d0723bfe.bmp)
    - Equalizer Coefficients
      - Presets allow a device to use one of 11 possible starting values to be used for the partner’s Transmitter coefficients when first training to the 8.0 GT/s data rate.
      - This is accomplished by sending EQ TS1s and EQ TS2s during training which gives a coarse adjustment of Tx equalization as a starting point.If the signal using  the  preset  delivers  the  desired  10‐12  error  rate,  no  further  training  is needed. But if the measured error rate is too high, the equalization sequence is used to fine tune the coefficient settings by trying different C‐1 and C+1 values and evaluating the result, repeating the sequence until the desired signal quality or error rate is achieved.
      - An 8.0 GT/s transmitter is required to report its range of supported coefficient values to its neighboring Receiver. There are some constraints on this:
        > - Device must support all 11 presets as listed
        > - Transmitters must meet the full‐swing VTX‐EIEOS‐FS signaling limits
        > Transmitters  may  optionally  support  the  reduced‐swing,  and  if  they  do they must meet the VTX‐EIEOS‐RS limits
        > Coefficients  must  meet  the  boost  limits  (VTX‐BOOST‐FS  =  8.0  dB  min,  VTX-BOOST‐RS = 2.5 dB min) and resolution limits (EQTX‐DOEFF‐RESS = 1/24 max to 1/63 min).
   - Link Power Management States
![新建位图图像 (8).bmp](attachments\88e0335e.bmp)
![新建位图图像 (9).bmp](attachments\f927d384.bmp)  
![新建位图图像 (10).bmp](attachments\9110a423.bmp)  
![新建位图图像 (11).bmp](attachments\282f69da.bmp)
![新建位图图像 (12).bmp](attachments\c94754bd.bmp)
