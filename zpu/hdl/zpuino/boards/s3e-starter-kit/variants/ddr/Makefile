PROJECT=s3e_eval_zpuino
PART=xc3s500e-fg320-4

# For bootloader
BOARD=S3E_EVAL
SIZE=32768
DEFINES="-DBOARD_ID=0x83011A00 -DBOARD_MEMORYSIZE=0x04000000"
ROOT=../../../..
all: ${PROJECT}_routed.bit ${PROJECT}_routed.bin

bootloader.vhd:
	$(MAKE) -C $(ROOT)/bootloader BOARD=$(BOARD) SIZE=$(SIZE) DEFINES=$(DEFINES)
	#cp $(ROOT)/bootloader/prom-generic-dp-32.vhd .
	cp $(ROOT)/bootloader/bootloader.vhd .
	
${PROJECT}.ngc: bootloader.vhd #prom-generic-dp-32.vhd
	mkdir -p xst/projnav.tmp/
	xst -intstyle ise -ifn ${PROJECT}.xst -ofn ${PROJECT}.syr
        
${PROJECT}.ngd: ${PROJECT}.ngc
	ngdbuild -intstyle ise -dd _ngo -nt timestamp \
	-uc ${PROJECT}.ucf -p ${PART} ${PROJECT}.ngc ${PROJECT}.ngd

${PROJECT}.ncd: ${PROJECT}.ngd
	map -intstyle ise -p ${PART} \
	-cm speed -detail -ir off -ignore_keep_hierarchy -pr b -timing -ol high -logic_opt on  \
	-o ${PROJECT}.ncd ${PROJECT}.ngd ${PROJECT}.pcf 

${PROJECT}_routed.ncd: ${PROJECT}.ncd
	par -w -intstyle ise -ol high -t 1 ${PROJECT}.ncd ${PROJECT}_routed.ncd ${PROJECT}.pcf

${PROJECT}_routed.bit: ${PROJECT}_routed.ncd
	bitgen -f ${PROJECT}.ut ${PROJECT}_routed.ncd

${PROJECT}_routed.bin: ${PROJECT}_routed.bit
	promgen -w -spi -p bin -o ${PROJECT}_routed.bin -s 1024 -u 0 ${PROJECT}_routed.bit

clean:
	@rm -rf ${PROJECT}.{ngc,ngd,ncd,_routed.ncd,pcf,bit,_routed.bit} prom-generic-dp-32.vhd bootloader.vhd
	$(MAKE) -C $(ROOT)/bootloader clean
