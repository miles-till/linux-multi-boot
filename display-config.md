# ddcutil

```sh
❯ ddcutil detect
Display 1
   I2C bus:  /dev/i2c-4
   DRM connector:           card1-DP-2
   EDID synopsis:
      Mfg id:               DEL - Dell Inc.
      Model:                DELL U2412M
      Product code:         41083  (0xa07b)
      Serial number:        PPNN15CU01TL
      Binary serial number: 808539212 (0x3031544c)
      Manufacture year:     2015,  Week: 53
   VCP version:         2.1

Display 2
   I2C bus:  /dev/i2c-5
   DRM connector:           card1-DP-3
   EDID synopsis:
      Mfg id:               DEL - Dell Inc.
      Model:                DELL S2721DGF
      Product code:         16857  (0x41d9)
      Serial number:        883PS83
      Binary serial number: 1263815244 (0x4b544a4c)
      Manufacture year:     2021,  Week: 17
   VCP version:         2.1
```

- `-b 5` selects I2C bus 5 `/dev/i2c-5`

```sh
❯ ddcutil -b 5 probe
...
   Feature: 60 (Input Source)
      Values (unparsed): 0F 11 12
      Values (  parsed):
         0f: DisplayPort-1
         11: HDMI-1
         12: HDMI-2
...
```

- `0x60` refers to input source feature
- `0x0f` selects input source

```sh
ddcutil -b 5 setvcp 0x60 0x0f # DP 1    Meta+Shift+1
ddcutil -b 5 setvcp 0x60 0x11 # HDMI 1  Meta+Shift+2
ddcutil -b 5 setvcp 0x60 0x12 # HDMI 2  Meta+Shift+3
```
