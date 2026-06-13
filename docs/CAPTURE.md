# Capturing the README demo media

The README references two files in this folder. Capture them in the Kali VM with
the firewall running, then commit them:

- `docs/demo-detection.gif` — a terminal recording of `tests/run.sh` (the green scorecard)
- `docs/gui-overview.png`   — a screenshot of the `sentinel-gui` control panel

Keep each file under ~10 MB so GitHub renders it inline.

---

## 1. GUI screenshot → `docs/gui-overview.png`

```bash
sudo apt-get install -y flameshot scrot      # either works
sudo systemctl start sentinel && sleep 40     # engines online
./run-gui.sh &                                # launch the panel
sleep 5

# Interactive (select the window, then save as docs/gui-overview.png):
flameshot gui

# …or non-interactive full screen after a 3s delay:
scrot -d 3 docs/gui-overview.png
```

Tip: open the **Overview** tab (live KPIs) or the **Downloads** tab (verdicts +
quarantine) before capturing — they read best in a still.

---

## 2. Detection-run GIF → `docs/demo-detection.gif`

Pick whichever tool you have. **Option A (VHS)** gives the cleanest, scripted result.

### Option A — VHS (recommended, fully scripted)
```bash
# install: https://github.com/charmbracelet/vhs  (needs ttyd + ffmpeg)
cat > docs/detection.tape <<'EOF'
Output docs/demo-detection.gif
Set FontSize 16
Set Width 1200
Set Height 760
Set Theme "Catppuccin Mocha"
Type "bash tests/run.sh"
Enter
Sleep 45s
EOF
vhs docs/detection.tape
```

### Option B — asciinema + agg
```bash
sudo apt-get install -y asciinema
asciinema rec docs/detection.cast -c "bash tests/run.sh"
# then convert (agg: https://github.com/asciinema/agg):
agg --font-size 18 docs/detection.cast docs/demo-detection.gif
```

### Option C — simplest (a still PNG instead of a GIF)
```bash
bash tests/run.sh                              # let it finish in a large terminal
scrot docs/demo-detection.png                  # screenshot the scorecard
# then point the README at the .png instead of .gif
```

---

## 3. (Optional) a fuzz-pass GIF → `docs/demo-fuzz.gif`
Same as Option A/B but record `bash tests/fuzz.sh` — the live per-input progress
ending in **`RESULT: PASS — survived every malformed input`** is a strong clip.

---

After capturing, the README image links resolve automatically. Trim/crop with any
editor; for GIF size, lower the resolution or font size in the tape/agg command.
