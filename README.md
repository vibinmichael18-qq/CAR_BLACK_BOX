# 🚗 Car Black Box

An embedded-systems project that mimics a vehicle **event data recorder ("black box")**. It runs on a **PIC18F4580** microcontroller. It shows live speed, gear and time on an LCD. It also logs every gear change and collision event to external EEPROM, so the data survives power loss. The log can be viewed on the LCD, downloaded over UART, cleared, and the clock can be set from a menu.

Built with MPLAB X IDE and the XC8 compiler.

---

## ✨ Features

- **Live dashboard** on a 16x2 character LCD showing speed, gear (N, 1-5, R) and real-time clock.
- **Speed input** read from a potentiometer via the on-chip ADC (channel 4). Speed is scaled to 0-100 and capped by the selected gear.
- **Event logging** to an external I2C EEPROM (24Cxx): every gear change and collision is stored with speed, gear and timestamp.
- **Collision event** flagged with a `C` marker in the log.
- **Circular log buffer** holding the 10 most recent events. The oldest entry is overwritten when the buffer is full.
- **Persistent storage**: log count and write index are saved in EEPROM and restored at boot.
- **Menu system** (View / Download / Set / Clear log) navigated with a matrix keypad.
- **Log download over UART** (9600 baud) to a PC terminal, with collision entries tagged.
- **Real-time clock** using a DS1307 over I2C, with an on-device "Set RTC" screen.

---

## 🧰 Hardware

| Component | Purpose | Interface |
|---|---|---|
| PIC18F4580 | Main microcontroller | - |
| 16x2 character LCD | Display | PORTD (data), RC0/RC1/RC2 (RW/RS/EN) |
| 12-key matrix keypad (4x3) | User input | PORTB (RB1-RB7) |
| Potentiometer | Simulates the accelerator and speed | ADC channel 4 (AN4) |
| DS1307 RTC | Timekeeping | I2C (address `0xD0`/`0xD1`) |
| 24Cxx external EEPROM | Non-volatile event log | I2C (address `0xA0`/`0xA1`) |
| USB-TTL / RS-232 module | Log download to PC | UART, RC6 (TX) / RC7 (RX) |

---

## 🎛️ Controls

| Key | Dashboard | Menu |
|---|---|---|
| `SW1` | Gear up (N, 1, 2, ... 5, R) | Scroll down / next entry |
| `SW2` | Gear down | Scroll up / previous entry |
| `SW3` | Log a **collision** event | - |
| `SW11` | Enter the menu | Select the highlighted option |
| `SW12` | - | Go back |

### Gear speed limits

| Gear | Max speed |
|---|---|
| N | 0 |
| 1 | 30 |
| 2 | 45 |
| 3 | 75 |
| 4, 5 | No cap (full scale) |
| R | 40 |

---

## 📋 Menu

1. **View log**: browse stored events on the LCD.
2. **Download log**: send all events to a PC terminal over UART.
3. **Set log** (set RTC): set the hours, minutes and seconds of the clock.
4. **Clear log**: reset the log counter and write pointer.

### Log entry format

Each event takes 16 bytes of EEPROM, starting at addresses `0x03, 0x13, 0x23, ... 0xA3`.

```
SPD G  HH:MM:SS C
 |  |     |     |
 |  |     |     +-- 'C' if collision, blank otherwise
 |  |     +-------- time from DS1307
 |  +-------------- gear (N, 1-5, R)
 +----------------- speed (000-100)
```

### EEPROM header

| Address | Content |
|---|---|
| `0x00` | Number of stored events (ASCII digit) |
| `0x01` | `'1'` once a log exists (validity flag) |
| `0x02` | Last-written index (ASCII digit) |

---

## 📁 Project structure

```
Project2.X/
├── main.c / main.h        # Entry point, dashboard, menu and gear/event logging
├── sub_modes.c            # View, download and clear log
├── Get_time.c             # RTC reading, time display, Set RTC screen
├── RTC.c / RTC.h          # DS1307 driver
├── eeprom.c / eeprom.h    # External EEPROM read/write over I2C
├── I2C.c / I2C.h          # I2C master driver
├── adc.c / adc.h          # ADC driver (speed input)
├── clcd.c / clcd.h        # 16x2 LCD driver
├── matrix.c / matrix.h    # Matrix keypad driver
├── uart.c / uart.h        # UART driver (log download)
├── timer.c                # Timer2 setup and ISR
├── Makefile
└── nbproject/             # MPLAB X project configuration
```

---

## 🚀 Getting started

### Requirements

- [MPLAB X IDE](https://www.microchip.com/mplab/mplab-x-ide)
- [XC8 compiler](https://www.microchip.com/mplab/compilers)
- Device pack: `PIC18Fxxxx_DFP`
- A programmer/debugger (PICkit 3/4 or similar)
- A serial terminal (PuTTY, Tera Term) for log download

### Build and flash

1. Open MPLAB X IDE and choose **File -> Open Project**, then select the `Project2.X` folder.
2. Confirm the device is **PIC18F4580** and the XC8 compiler is selected.
3. Click **Clean and Build**.
4. Connect your programmer and click **Make and Program Device**.

### Download the log to a PC

1. Connect the UART TX/RX pins to a PC through a USB-TTL or RS-232 adapter.
2. Open a serial terminal at **9600 baud, 8 data bits, no parity, 1 stop bit**.
3. On the device, go to **Menu -> Download log**.

Sample output:

```
SPD G   RTC
045 2  10:32:15
075 3  10:32:20
030 1  10:32:41  Collision
```

---

## 🔧 How it works

- On boot the firmware initialises the ADC, LCD, I2C, keypad, DS1307 and UART, then restores the log state from EEPROM.
- The main loop polls the keypad, reads the RTC and refreshes the display.
- The ADC value (0-1023) is scaled to 0-100 and clamped to the gear's limit.
- A gear change or collision key press writes a 16-byte record into the next slot of the circular buffer and updates the header bytes.

---

## 🛠️ Possible improvements

- Use hardware interrupts (e.g. an accelerometer) for automatic collision detection.
- Increase the log depth beyond 10 entries using the full EEPROM capacity.
- Store dates as well as times.
- Add a password to protect the clear and download actions.

---

## 📄 License

Add a license of your choice (for example MIT) to this repository.

## 👤 Author

Your name: [GitHub profile](https://github.com/your-username)
