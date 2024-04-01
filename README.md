# Data-Acquisition-system-for-Beam-profile-measurement
The following project is a data acquisition system designed and developed for taking the data signals of the ion beam from the accelerator (through HARP sensor) to study various characteristics like beam intensity,emittance and space charge effects.

Structural Diagram of VHDL Code dhowing different modules
![image](https://github.com/TripleS1404/Data-Acquisition-system-for-Beam-profile-measurement/assets/99711972/99c1e509-6c86-48c6-a328-b4cec2160300)

Working description of the full VHDL Code

a. The Local Controller1 interfaces with 3 other modules 
maincontoller,SHlatch4channel, AD7824interface and is initially in the default state 
Idle state. It receives ch_addr,gr_addr,data_group signals from the main controller 
or externally and waits for acq_all and acq_sel. If both of them are either 0 or 1 at 
the same time, the FSM of Local Controller1 continues in the present state. But if 
acq_all and acq_sel are of not the same value,it indicates that either all the four are 
to be acquired or the selected channel, whose address is indicated by ch_addr, is to 
be acquired. Then Local Controller1 sets ch_sel bus with ch_addr value and asserts 
enable to 1 to enable SHlatch4channel. But if acq_all='1' and acq_sel='0', then 
latch_all is set at 1. 

b.The FSM of SHlatch4channel on receipt of enable as '1', starts the data acquisition 
process. The SHlatch4channel interfaces with 4 LF398 S/H amplifiers. It controls 4 bit 
“latched” signal , each bit of which is used as LOIGC input to the S/H 
amplifier.Normally the S/H amplifiers are in sample mode. If latch_all received is 1, 
the module puts all the four S/H amplifiers in hold mode through assigning of 
appropriate digital code to ‘latched’ signal so that analog input signals from the IVC 
are sampled and held. If latch_all received is 0, the module puts only one of the four 
S/H amplifiers in hold mode, depending on the ch_sel address set by the local 
controller1. 

c.The SHlatch4channel sends the feedback signals “sampled_addr1”, 
“sampled_select” and “sampled_all” to the Local_controller1.

d. The Local_controller1 checks for the status of two signals sampled_all, 
sampled_select. If both of them are either 0 or 1 at the same time, the signal 
ch_addr2adc is set to high impedance state ("ZZ") and the Local_controller1 FSM 
waits in the present state. But for sampled_all='1' and sampled_select='0') or 
(sampled_all='0' and sampled_select='1'), it indicates that SH4channel has 
completed sampling process and so ch_addr2adc is assigned with the value of 
sampled_addr1 received from SH4channel and Local_controller1 FSM switches to 
startadc state.

e. Now Local_controller1 FSM asserts the start signal as '1', sends it to 
AD7824interface module and waits for data_ready to become '1'.

f.In response, the FSM of AD7824 interface module goes to set_adc_ch state and sets 
the channel address of the ADC. It continues in this state for the next 5 clock cycles 
to allow the settling time for the ADC channel selection process. On completion of 5 
clock cycles, the FSM interfaces with AD7824 by switching through start_conv, 
start_conv, read_conv states. When FSM enters start_conv state, the A to D 
conversion is started by setting ncs='0' and nrd='0'.The value of sampled _ addr is 
assigned to acq_addr. The data_ready is set as '0' indicating that no data is available 
for Local_controller1 to read. When the FSM is in_conv state, the nrd and nINT 
signals are continusoly checked whether the acquisition is complete or not. If the 
signals nrd ='0' and nINT='1' or if nrd='1' and nINT='1', they indicate that ADC is in 
the middle of conversion and so FSM continues in the present state ie in_conv state. 
Otherwise if (nINT='0')) and (nrd='0' or ‘1’), it indicates that the conversion is 
complete and a valid conversion result is available to be read. This prompts the FSM 
to switch to read_conv state. In read_conv state, data_ready is set as '1' indicating 
that the data is ready for Local_controller1 to read and the FSM goes to end_conv 
state. In end_conv state, the status of adc_data_read is continuously checked. If it is 
'0', then FSM continues in end_conv state. Otherwise the status of various signals 
are set as data_ready<='0', ncs<='1', nrd<='1' and the FSM switches to 
waitforacq state thus completing one data acquisition cycle.

g.On receiving the data ready as 1, the Local controller1 FSM checks the status of local 
controller2 whehter it is busy or not through the signal lc2_sts. If the signal lc2_sts is 
'0', the FSM waits. Otherwise, the lc1ch_addro, lc1dat_gro, lc1gr_addro are assigned 
with values of the signals acq_addr, data_group, gr_addr respectively and FSM 
switches to idle state.Also the signals read_adc_data, adc_data_readsts are set to '1'.

h.This marks the relinguishing of the control to local controller2 to proceed with 
processing of the ADC converted results. Now local controller1 is free to start 
another A to D conversion. Thus local conttoller1, local controller2 and local 
controller 3 handle different phases of data acquisition by parallel processing and 
pipelining. 

i.On being reset, the FSM of local controller2 waits in the default state of idle and sets 
lc2_ready as '1' to indicate its readiness. The FSM continues in the idle state as long 
as the adc_data_readsts sent by the local controller1 is '0'. Otherwise it asserts the 
load to '0' for a period of 5 clock cycles to the serinpo module.

j.The FSM of serinpo, on being reset, waits in idle state until local controller2 sets load 
as '0'. Once the load becomes 0, the FSM of serinpo indicates the status of ADC data 
loading into the shift register by setting load_done1 at 1. Also it sets sloclk_inhibit as 
0 prohibiting serial shifting of the data.

k.The FSM of local controller2 monitors load_done1 and asserts data_shift to '1' when 
the serinpo sets load_done1 at 1.

l.If data_shift becomes '1’, the FSM of serinpo start serial shifting of the data by 
enabling the serial clock and feeding it to the shift registers through setting of 
sloclk_inhibit at 0. The serial data is shifted from the shift register into the serinpo at 
every positive edge of the clock1. The shifting continues for 8 clock cycles. A the end 
of 8 clock cycles, the sloclk_inhibit is set at 1 to disable further serial shifting and the 
FSM sets data_shifted at 1 to indicate successful shifting of the 8 bit data from shift 
register to the serinPO.

m.The FSM of local controller2, on “data_shifted” becoming 1 sets latch_data at '1'.

n.The FSM of serinpo, on receiving latch_data as 1, latches the 8 bit data, assigns it to 
the output signal parallel_data, and sets data_latched as '1'.

o.The FSM of local controller2 also sends ch_addr, data_group and gr_addr it 
received from local controller1 to serinpo module.

p.The FSM of local controller2 sends ch_addrin_lc2, dat_grin_lc2, gr_addrin_lc2 
signals to local controller 3. With this local controller3 takes over to further process 
the parallel data received. The local controller2 is now free to do serial to parallel 
conversion of new data from ADC.

q.In the idle state, the FSM of LC3 sets ch_addr2mem,dat_gr2mem,gr_addr2mem 
with corresponding data received from the LC2 and waits for the write_data to 
become ‘1’. It also indicates its readiness to the LC2 by setting write_datasts as ‘1'. 
When the LC2 sets write_data='1', the LC3 switches to mem_wr state and waits for 2 
clock cycles during which it prompts rams_dps module to save the parallel data from 
serinpo module in the memory of rams_dps. During this time the LC3 sets its status 
as busy by setting write_datasts as '0'. It then switches to another acqdone state to 
indicate ‘acquisition complete’ and its readiness to the LC2 for another memory 
write by setting write_datasts as ‘1'.

r.rams_dps module carries out two functions of memory write and memory read. For 
memory write, memory address and control ‘ena_wr’ are received from the LC3. For 
memory read, the read address and ena_rd are to be provided from an external 
controller. 


