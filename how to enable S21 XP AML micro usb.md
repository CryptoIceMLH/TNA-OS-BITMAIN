# Antminer S21 XP — Enable Micro USB Port (AML Control Board)

Stock Bitmain firmware released after September 2025 has the micro USB port disabled.
This guide re-enables it so you can downgrade firmware and install TNA-OS.

---

## What You Need

- AML control board + 12V power supply
- Windows PC or laptop
- USB-to-UART adapter with drivers installed (e.g. Silicon Labs CP2102)
- PuTTY (or any serial terminal)
- jumper or dupon wire to short JP1

---

## Step 1 — Wire Up the UART Adapter

Connect your USB-to-UART adapter to the control board:

| Adapter | Board |
|---------|-------|
| GND | GND |
| RXD | LINUX_TX |
| TXD | LINUX_RX |

Plug the adapter into your PC and note the COM port number (e.g. COM7) from Device Manager.

---

## Step 2 — Open PuTTY

- Connection type: **Serial**
- COM port: your port (e.g. COM7)
- Baud rate: **115200**
- Everything else: leave as default

---

## Step 3 — Verify Wiring

Power on the board. You should see the boot log scrolling in PuTTY.
If you see nothing, check your TX/RX wiring — they are often swapped.

Power the board back off once confirmed.

---

## Step 4 — Get the U-Boot Prompt

This is the critical step. You need to interrupt the boot sequence at exactly the right moment.

1. Make sure the PuTTY window is open and active
2. Short **JP2** with jumper / dupon wire
3. Power on the board
4. **The instant the first boot line appears** — release JP1 and tap the spacebar a few times

You should see U-Boot start loading the ENV and drop to a prompt:

```
axg_s400_v1_sbr#
```

**If it doesn't work first try** — power off, power on again, and briefly short JP2  a couple of times during boot. This forces the board into U-Boot mode.

---

## Step 5 — Enable the USB Port

At the `axg_s400_v1_sbr#` prompt, run these commands in order:

```
saveenv
setenv bitmain_usb_switch 1
saveenv
```

The first `saveenv` initialises the NAND env partition if it hasn't been written before.
`setenv` enables the USB port. The second `saveenv` writes it permanently to NAND.

---

## Step 6 — Reboot

Power cycle the board. The micro USB port is now enabled.

You can now plug in a FAT32 USB drive with the downgrade firmware and the board will detect it on boot.

---

## Troubleshooting

**Seeing `nand init failed` or `saveenv` errors?**
Run the commands in this exact order — the first `saveenv` is required to initialise the partition before you can write to it:
```
saveenv
setenv bitmain_usb_switch 1
saveenv
```

**Board not dropping to U-Boot prompt?**
- Make sure PuTTY is the active window when you tap spacebar
- Try shorting JP2 slightly earlier or later during power-on
- A brief short is enough — do not hold it down

If you fancy supporting the my work head over to https://www.molonlabe.holdings/#funding
