# Quick guide: use this repo to add Rocky Linux to the PXE menu

This file is intentionally minimal — steps to add a Rocky Linux ISO and expose it via the existing PXE server.

1. Add Rocky to URLs (prefer custom file)

- Edit `c2-custom-url` and add:

```bash
ROCKY_X64=rocky-x64
ROCKY_X64_URL=https://download.rockylinux.org/pub/rocky/10/isos/x86_64/Rocky-10.0-x86_64-minimal.iso
ROCKY_X64_SUM_TYPE=sha256
```

2. Expose the image to the orchestrator

- Edit `c2-custom-handle` and add a handler line (no $ prefix):

```bash
handle_item '+' iso ROCKY_X64;
```

3. (Optional) Add a tiny PXE menu fragment

- Edit `c2-custom-menu` and include a menu block using the repository markers:

```
#========== BEGIN ==========
LABEL rocky-x64
    MENU LABEL Rocky Linux 9 (x86_64)
    KERNEL /srv/iso/rocky-x64/vmlinuz
    APPEND initrd=/srv/iso/rocky-x64/initrd.img boot=live
#=========== END ===========
```

Adjust the KERNEL/INITRD paths to match how the ISO is exposed under `/srv/iso` after `p2-update`. If unsure, skip this step — `p2-update` will often create sensible defaults.

4. Run the update phase to download and expose the ISO

- If the PXE server is already installed and configured, run:

```bash
bash run.sh update
```

- If this is a fresh Pi, follow the three-run sequence in `README.md`: `bash run.sh` → reboot → `bash run.sh` → reboot → `bash run.sh` (or force phases: `bash run.sh install|setup|update`).

5. Validate the download URL first (optional):

```bash
curl -I -s -o /dev/null -w "%{http_code}" "https://download.rockylinux.org/pub/rocky/9/isos/x86_64/Rocky-9.4-x86_64-minimal.iso"
```

Expect 200 or a redirect code (30x).

Notes

- Prefer editing `c2-custom-*` files to keep your changes separate from upstream.
- Handlers expect the variable NAME without `$` when referenced (`handle_item '+' iso ROCKY_X64;`).
- Disk roots used by the scripts: `/srv/iso`, `/srv/img`, `/srv/nfs`; PXE menu root: `/srv/tftp/menu-bios`.

If you'd like, I can add a tested example entry (exact URL + checksum) and a minimal PXE menu tuned for Rocky. Tell me whether you want the menu fragment created automatically or prefer to craft it manually.
