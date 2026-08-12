# BC-250 BIOS

BIOS images for the **ASRock BC-250**, with **UNLOCK** builds that open up the
hidden **AMD CBS** menus. The headline feature is real **GDDR6 memory clock and
timing control** — something no other public BC-250 BIOS gives you.

## What's in here

| File | What it is |
|---|---|
| `BC250_3.00_STOCK.ROM` | Stock P3.00, untouched. |
| `BC250_3.00_CHIPSETMENU.ROM` | Segfault's well-known P3.00 mod — VRAM + NBIO menu. The community standard. |
| `BC250_3.00_UNLOCK.ROM` | P3.00 with the **full CBS unlock**. |
| `BC250_5.00_STOCK.ROM` | Stock P5.00, untouched. |
| `BC250_5.00_UNLOCK.ROM` | P5.00 with the **full CBS unlock**. |
| `AfuEfix64.efi`, `Flash.nsh`, `amdvbflash.efi`, `EFI/` | The EFI shell flashing kit. |

### Hashes (SHA256)

```
48fbe5d366e6a56e2fdffdca848426216ba1f083610dab63db89d2f4e6c940b5  BC250_3.00_CHIPSETMENU.ROM
07595ca3aecf8a4caa28a397b5298f3946a1b769f87b16f67adc369c3f69045c  BC250_3.00_STOCK.ROM
dc0a12b584459475a6262f264110fcee9826fdc789f17496ad92f4741c725d11  BC250_3.00_UNLOCK.ROM
0d6f136cb120cf3b2de26d5c4d7f255604fdbf4b9442af5ba55419b95b89aa82  BC250_5.00_STOCK.ROM
e604b284b447b8341cde80d1590afb76a4b2b3bfbec40a0acc12d88322f348c1  BC250_5.00_UNLOCK.ROM
```

Run `sha256sum` and check before flashing. Every image is exactly 16 MiB.

## What UNLOCK actually unlocks

The CBS menus are already in the firmware — they're just orphaned, sitting in
the binary with no menu linking to them. UNLOCK adds a few references on the
**Chipset** page so you can finally reach them.

You get **4 new entries**, opening about a dozen forms:

- **Vh Common Options** — Core Performance Boost, C-states, Downcore, SMT…
- **DF Common Options** — memory interleaving and related fabric settings.
- **NBIO Common Options** — the iGPU (UMA / VRAM split), IOMMU, PCIe, USB.
- **UMC Common Options** — and this is the one you came for: **GDDR6 timings**.

### The GDDR6 part (why this exists)

Under `UMC → GDDR6 → DRAM Timing Configuration → "I Accept"` you'll find:

- **Memory clock** in 22 steps, from 125 up to 2000 MHz (plus Auto).
- **Primary timings**: Tcl, Tras, Trcdrd/wr, Trp, Trc, Trrd, Trtp, TFaw…
- **Refresh**: Tref, TrfcPb, Trfc.

This matters because the BC-250 is bottlenecked by memory bandwidth. The
famous 40-CU unlock barely moves games (~4 %) for exactly that reason.
If you want real gains, memory clocks and timings are the lever that helps.

### What it doesn't do

To be clear about the limits:

- **No Resizable BAR.** There simply isn't one in this firmware — nothing to
  turn on. `Above 4G Decoding` already exists under *Advanced*, untouched.
- **No PPT or forced CPU frequency.** Those live in SMU menus that aren't
  linked here. For CPU-side tuning, use `bc250_smu_oc` instead.
- **Nothing is invented.** UNLOCK only makes existing menus reachable — it
  doesn't add options that weren't already compiled in.

The only thing it removes is the empty `North Bridge` page (just placeholder
text). `Memory Configuration` is re-linked, so nothing is actually lost. South
Bridge, fan control, SATA, USB — all preserved.

## Which one should I flash?

- Just want VRAM / the dynamic 512 MB? → `BC250_3.00_CHIPSETMENU.ROM`.
- Want to tune GDDR6 and poke the full CBS tree? → a `*_UNLOCK.ROM`.
- Broke something? → flash back a `*_STOCK.ROM`.

If you've never flashed this board before, start with the CHIPSETMENU image.
It's proven on hardware and confirms your flashing setup works before you move
on to an UNLOCK build.

## Flashing over USB (EFI shell)

1. Format a USB stick as FAT32. Unzip the flashing kit and copy everything to
   the root: `AfuEfix64.efi`, `Flash.nsh`, `amdvbflash.efi`, and the **`EFI`
   folder** (the board needs that folder to boot into the shell).
2. Copy your chosen ROM to the root as well, then point `Flash.nsh` at it:
   open `Flash.nsh` and put your filename in the command, e.g.
   `AfuEfix64.efi BC250_5.00_UNLOCK.ROM /p /b /n /k /x /rlc:e`
3. Unplug all SSDs so the board falls through to the EFI shell, then boot.
4. At the `Shell>` prompt: `blk0:` then `Flash.nsh`.
5. **Don't interrupt it.** If it looks stuck, wait 15 minutes. Cutting power
   mid-flash is how boards get bricked.
6. Power off, pull the stick, and **clear the CMOS** (CR2032 battery out for a
   minute, press power a few times, put it back). Skip this and your settings
   won't stick.
7. Then set: Chipset → GFX → iGPU **Forces**, UMA Mode **UMA_SPECIFIED**, Frame
   Buffer **512 MB**; NB Configuration → IOMMU **Disabled**. Save & exit.

### With a hardware programmer (backup / recovery)

This is the only way back if the board won't POST:

```bash
sudo flashrom -p ch347_spi                        # detect
sudo flashrom -p ch347_spi -r backup_stock.bin    # back up first, always
sudo flashrom -p ch347_spi -w BC250_5.00_UNLOCK.ROM
```

Target the **`BIOS_A1`** chip (16 MB). **Never** touch `SIO1_R` — that's the
512 KB SuperIO, and flashing it kills the fan control. Avoid the cheap
black-PCB CH341A programmers: many output 5 V logic even in 3.3 V mode and can
fry the chip. Use a CH347 or a verified 3.3 V unit.

## Safety

Don't flash an UNLOCK image without a programmer and a known-good backup in
hand. "Exposed" doesn't mean "safe" — wrong memory timings or chipset values
can stop the board from POSTing. Change one thing at a time, and know how
you'll recover before you start. And never use Smokeless_UMAF for VRAM on this
board — that's a permanent brick risk, and it's exactly what these images exist
to avoid.

Oh, and double-check the 8-pin PCIe power wiring before the first power-on.
Reversed polarity will kill the board.

## Credits

- **Segfault / TuxThePenguin0** — the original reverse engineering, the
  chipset-menu mod, the SPI pinout ([GitLab](https://gitlab.com/TuxThePenguin0/bc250-bios/))
- **elektricM & the BC-250 community** — the [docs](https://elektricm.github.io/amd-bc250-docs/)
  and the verified hash table
- **kenavru** — the EFI flashing kit
