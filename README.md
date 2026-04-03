# SJGAM M17 (W12-V04)
Утилиты для SJGAM M17 с платой W12-V04 и всякие справки.

### Тех. характеристики консоли:

- **Процессор:** Rockchip RK3126C - 4 ядра Cortex-A7 - видеоядро Mali-400 MP2.
- **Оперативная память:** 256MB DDR3 - Samsung K4B1G0846F-HCH9 - 2 чипа по 128MB DDR3, работающих в 1333 МГц.
- **Встроенная память (eMMC):** Toshiba THGBMDG5D1LBAIL - 4 GB.
- **Дисплейный модуль:** матрица NoName GV043WXY002LA0703SW - 40 pin - 480x272 - 4.3".
- **Аккумулятор:** Li-Polymer LB 405267 - 2000mAh - 3.85V - 7.7Wh.
- **Контроллер питания:** AP5056.
- **Аудиоусилитель:** MIX2018A.
- **Порты:** USB Type-C (OTG включен в device-tree и на uboot и на boot, но по-моему, не разведен), TRS 3.5mm (Mini Jack в простонародии, вывод звука) и слот microSD (пока не известен макс. объем, который эта консолька скушает).
- **Управление:** механическое (кнопки громкости + / -, кнопки start / select, тумблер консоли On / Off, два аналоговых стика с кнопками R3 / L3, шифты L1 / R1 / L2 / R2, кнопки D-Pad и A / B / Y / X).
- **Вывод звука:** 1 плёночный динамик.

### Структура встроенной памяти eMMC:

| NO | LBA | Size | Name |
|:---|:---|:---|:---|
| 01 | `0x00004000` | `0x00002000` | **uboot** |
| 02 | `0x00006000` | `0x00002000` | **trust** |
| 03 | `0x00008000` | `0x00002000` | **misc** |
| 04 | `0x0000a000` | `0x00010000` | **boot** |
| 05 | `0x0001a000` | `0x00010000` | **recovery** |
| 06 | `0x0002a000` | `0x00010000` | **backup** |
| 07 | `0x0003a000` | `0x000c8000` | **oem** |
| 08 | `0x00102000` | `0x00100000` | **rootfs** |
| 09 | `0x00202000` | `0x0055dfdf` | **userdata** |

- uboot и trust: Типовые загрузчики RockChip, ОБЯЗАТЕЛЬНЫ!!!.
- - trust: содержит всякие ключи и подписи, которые не менять, не корректировать не нужно, оставляем как есть.
- - uboot: отвечает за инициализацию начинки (RAM, ROM и т.п.). Имеет device-tree, в котором максимум то, что можно поменять, так это порт UART в который будет выводится лог с консоли.

- misc, backup, oem, recovery: разделы пустышки или файлопомойки, которые роли в работе особо не играют!
- - misc: не открывается ничем, пустой полностью, тупо размечается как раздел, и изменений в нём не происходит.
- - backup: такой же случай, как с misc.
- - recovery: содержит всё, что в boot, содержит device-tree, но и имеет ramdisk с busybox от какой то левой консоли, не используется, само рекавери никак не запускается, тоже просто ради "галочки".
- - oem - имеет volctrl, отвечающий за звук, парситься в /etc/init.d/ и ещё по-моему в папке bin на rootfs. Можно переместить на любой другой раздел и перепарсить, в остальном раздел отвечает за некий test mode китайской бусибокс системы (грузит видео и аудио с раздела для проверки), который также не нужен.

- boot: бут как в Android, содержит device-tree, ramdisk отсутствует! Всем управляет busybox из rootfs. Грузит систему напрямую (rootfs), логика через /etc/init.d/. Если вставлена карта памяти - все равно грузится rootfs, но rootfs передает управление системе на карте памяти (em_ui.sh), если карты памяти нет, грузится /usr/bin/game, китайский GUI с играми, в качестве эмулятора выступает RetroArch.

- rootfs: Содержит систему Linux, собранную через Buildroot 2018.02-rc3. По сути ядро от которого большинство работает. SquashFS, ещё содержит VLC для test mode, сборная солянка вообщем.

- userdata: ОБЯЗАТЕЛЕН!!! Он r/w, в него скидываются всякие конфиги ретроарча с rootfs, без этого раздела запуска не ждите.

### Режим "LOADER"

Вход осуществляется так: выключаем консоль, вставляем USB Type-C кабель в консоль и в ПК/Ноутбук, зажимаем кнопку **-** на консоли и включаем её. 

### Режим "MASKROM"

Вход осуществляется в двумя способами: 

- 1: Выключаем консоль, разбираем, отключаем аккумулятор и динамик, переводим выключатель в позицию **On**, вставляем USB Type-C кабель в консоль, разьем USB держим возле ПК/Ноутбука (не втыкаем пока что), замыкаем [Test Point](Photos/testpoint.png) и втыкаем кабель. Вы в MASKROM. Также, как я, можно вывести кнопку для MASKROM (убедитесь, что кнопка имеет сопротивление в районе 1 Ома!!!).

- 2: Переход из под режима LOADER через кнопку в условном RKDevTool.

### Что такое режим "MASKROM"?

- Это самый низкоуровневый режим. Он зашит прямо в железо процессора и его невозможно стереть или повредить.
  
- Как это работает: При включении консоли процессор первым делом ищет загрузчик (u-boot) в eMMC. Если память пуста, повреждена или специально замкнут [Test Point](Photos/testpoint.png), процессор не находит код для запуска и переходит в MASKROM.
  
- Зачем нужен: Это «режим последней надежды». В нем устройство определяется компьютером как Rockusb Device, и через специальные утилиты (типа RKDevTool, RKDevelopTool или RKAndroidTool) можно залить прошивку на абсолютно чистую память.

### Что такое режим "LOADER"?

- Это штатный режим прошивки, когда первичный загрузчик уже работает, но он не грузит систему, а ждет команд от компьютера.

- Как это работает: В этом режиме работает микропрограмма-посредник (miniloader или uboot), которая умеет общаться с USB-портом и записывать данные в нужные разделы памяти.

- Зачем нужен: Это основной способ обновления прошивки вручную. В лоадере можно прошивать отдельные разделы (например, только recovery или только boot), не затирая всё остальное.

- Отличие от MaskRom: В Loader устройство попадает, если загрузчик в памяти исправен. Если ты стер память под ноль — в Loader ты не попадешь, только в MaskRom.

### Конфигурация UART порта: она устанавливается в device-tree из uboot, т.е. выбор порта. 

rk-kernel.dts из uboot.img:
```
/dts-v1/;

/ {
	#address-cells = <0x01>;
	#size-cells = <0x01>;
	compatible = "rockchip,rk3126-evb", "rockchip,rk3126";
	rockchip,sram = <0x01>;
	model = "Rockchip RK3126 Evaluation board";

	chosen {
		stdout-path = "/serial2@20068000";
	};

	aliases {
		serial1 = "/serial1@20064000";
		serial2 = "/serial2@20068000";
		mmc0 = "/dwmmc@1021c000";
	};

	nandc@10500000 {
		compatible = "rockchip,rk-nandc";
		reg = <0x10500000 0x4000>;
		interrupts = <0x00 0x12 0x04>;
		nandc_id = <0x00>;
		clocks = <0x03 0x43 0x03 0x1c5>;
		clock-names = "clk_nandc", "hclk_nandc";
		status = "okay";
		u-boot,dm-pre-reloc;
	};

	dmc@20004000 {
		compatible = "rockchip,rk3128-dmc", "syscon";
		reg = <0x00 0x20004000 0x00 0x1000>;
		u-boot,dm-pre-reloc;
	};

	clock-controller@20000000 {
		compatible = "rockchip,rk3128-cru";
		reg = <0x20000000 0x1000>;
		rockchip,grf = <0x05>;
		#clock-cells = <0x01>;
		#reset-cells = <0x01>;
		u-boot,dm-pre-reloc;
		phandle = <0x03>;
	};

	serial1@20064000 {
		compatible = "rockchip,rk3128-uart", "snps,dw-apb-uart";
		reg = <0x20064000 0x100>;
		interrupts = <0x00 0x15 0x04>;
		reg-shift = <0x02>;
		reg-io-width = <0x04>;
		clock-frequency = <0x16e3600>;
		clocks = <0x03 0x4e 0x03 0x156>;
		clock-names = "baudclk", "apb_pclk";
		dmas = <0x09 0x04 0x09 0x05>;
		#dma-cells = <0x02>;
		u-boot,dm-pre-reloc;
	};

	serial2@20068000 {
		compatible = "rockchip,rk3128-uart", "snps,dw-apb-uart";
		reg = <0x20068000 0x100>;
		interrupts = <0x00 0x16 0x04>;
		reg-shift = <0x02>;
		reg-io-width = <0x04>;
		clock-frequency = <0x16e3600>;
		clocks = <0x03 0x4f 0x03 0x157>;
		clock-names = "baudclk", "apb_pclk";
		dmas = <0x09 0x06 0x09 0x07>;
		#dma-cells = <0x02>;
		u-boot,dm-pre-reloc;
		status = "okay";
	};

	usb2-phy {
		compatible = "rockchip,rk3128-usb2phy";
		reg = <0x17c 0x0c>;
		rockchip,grf = <0x05>;
		clocks = <0x03 0x8e>;
		clock-names = "phyclk";
		#clock-cells = <0x00>;
		clock-output-names = "usb480m_phy";
		#phy-cells = <0x01>;
		status = "okay";
		u-boot,dm-pre-reloc;
		phandle = <0x17>;

		otg-port {
			#phy-cells = <0x00>;
			interrupts = <0x00 0x23 0x04 0x00 0x33 0x04 0x00 0x34 0x04>;
			interrupt-names = "otg-bvalid", "otg-id", "linestate";
			status = "okay";
			u-boot,dm-pre-reloc;
		};
	};

	usb@10180000 {
		compatible = "rockchip,rk3128-usb", "rockchip,rk3288-usb", "snps,dwc2";
		reg = <0x10180000 0x40000>;
		interrupts = <0x00 0x0a 0x04>;
		dr_mode = "otg";
		g-use-dma;
		hnp-srp-disable;
		phys = <0x17 0x00>;
		phy-names = "usb";
		status = "okay";
		u-boot,dm-pre-reloc;
		vbus-supply = <0x18>;
	};

	dwmmc@1021c000 {
		compatible = "rockchip,rk3128-dw-mshc", "rockchip,rk3288-dw-mshc";
		reg = <0x1021c000 0x4000>;
		max-frequency = <0x8f0d180>;
		interrupts = <0x00 0x10 0x04>;
		clocks = <0x03 0x1cb 0x03 0x47 0x03 0x75 0x03 0x79>;
		clock-names = "biu", "ciu", "ciu-drv", "ciu-sample";
		bus-width = <0x08>;
		default-sample-phase = <0x9e>;
		num-slots = <0x01>;
		fifo-depth = <0x100>;
		resets = <0x03 0x53>;
		reset-names = "reset";
		status = "okay";
		u-boot,dm-pre-reloc;
		fifo-mode;
	};

	syscon@20008000 {
		compatible = "rockchip,rk3128-grf", "syscon";
		reg = <0x20008000 0x1000>;
		u-boot,dm-pre-reloc;
		phandle = <0x05>;
	};
};
```

Тут нам важны несколько строк, а именно:
```
chosen {
		stdout-path = "/serial2@20068000";
	};
```
и
```
serial1@20064000 {
		compatible = "rockchip,rk3128-uart", "snps,dw-apb-uart";
		reg = <0x20064000 0x100>;
		interrupts = <0x00 0x15 0x04>;
		reg-shift = <0x02>;
		reg-io-width = <0x04>;
		clock-frequency = <0x16e3600>;
		clocks = <0x03 0x4e 0x03 0x156>;
		clock-names = "baudclk", "apb_pclk";
		dmas = <0x09 0x04 0x09 0x05>;
		#dma-cells = <0x02>;
		u-boot,dm-pre-reloc;
	};

	serial2@20068000 {
		compatible = "rockchip,rk3128-uart", "snps,dw-apb-uart";
		reg = <0x20068000 0x100>;
		interrupts = <0x00 0x16 0x04>;
		reg-shift = <0x02>;
		reg-io-width = <0x04>;
		clock-frequency = <0x16e3600>;
		clocks = <0x03 0x4f 0x03 0x157>;
		clock-names = "baudclk", "apb_pclk";
		dmas = <0x09 0x06 0x09 0x07>;
		#dma-cells = <0x02>;
		u-boot,dm-pre-reloc;
		status = "okay";
	};
```
Перед нами две конфигурации портов UART, serial1 и serial2:

[UART 1 (serial 1)](Photos/uart1.jpg) расположен позле TEST POINT, и как видим, добрый товарищ китаец его не включил!

Для того, чтобы его включить, перемещаем опцию -
```
status = "okay";
```
Из serial2@20068000 в serial1@20064000.

Также, можно запаятся на так называемый [UART 2 (serial 2)](Photos/uart2.jpg) (фото [trash-9](https://4pda.to/forum/index.php?showtopic=1078486&view=findpost&p=141322246))
