# Dual-Booting a Lenovo Legion Go 2 with Windows + Bazzite + rEFInd (Secure Boot ON)

> This is the guide I wish I had found.

I wanted a Lenovo Legion Go 2 that could:

- Run **Windows** (for Call of Duty and other anti-cheat titles)
- Run **Bazzite** (for a SteamOS-style Linux gaming experience)
- Use **rEFInd** as the boot menu
- Keep **Secure Boot enabled**
- Avoid custom PK/KEK/OpenSSL key-signing hell

This post documents the **correct order**, the **correct boot chain**, and the **exact steps** required to make this work cleanly.

If you follow this guide **in order**, you will avoid every trap I fell into.

---

## The Mental Model (Read This First)

Secure Boot only works if your Linux boot chain looks like this:

UEFI firmware  
→ shimx64.efi (Microsoft-signed)  
→ rEFInd (trusted via MOK)  
→ Linux kernel

Windows boots separately via:

UEFI firmware  
→ Windows Boot Manager

**Key rule:** rEFInd must never be launched directly under Secure Boot. It must always be launched *through shim*.

---

## Part 1 — Prepare Windows

### Shrink the Windows partition

1. Open **Disk Management**
2. Shrink the main Windows partition
3. Leave the space **unallocated**

Do not manually create partitions.

---

## Part 2 — Install Bazzite (Secure Boot OFF temporarily)

Enter BIOS (Volume + Power) and disable Secure Boot.

Boot the Bazzite installer and install into the unallocated space.  
Reboot and confirm both OSes boot via BIOS boot menu.

---

## Part 3 — Install rEFInd (Secure Boot OFF)

Boot into Bazzite and run:

sudo dnf install refind  
sudo refind-install

Reboot and confirm rEFInd loads and boots both OSes.

---

## Part 4 — Reset Secure Boot Keys (Recommended)

Enter BIOS and select **Restore Factory Keys**.

---

## Part 5 — Enable Secure Boot

Re-enable Secure Boot in BIOS.

Boot Bazzite via **Fedora** (shimx64.efi).

---

## Part 6 — Reinstall rEFInd via shim

In Bazzite:

sudo refind-install --shim /boot/efi/EFI/fedora/shimx64.efi

---

## Part 7 — MOK Enrollment

On reboot, enter MokManager:

Enroll key from disk → SYSTEM_DRV → EFI/refind/keys/refind.cer  
Enroll key → Yes → Reboot

---

## Part 8 — Verify Secure Boot

In Windows PowerShell:

Confirm-SecureBootUEFI

Expected output:

True

---

## Part 9 — Cleanup and Backup

Prune duplicate EFI entries using efibootmgr.

Backup EFI:

sudo tar -C /boot/efi -cvpzf ~/efi-backup-$(date +%Y%m%d).tar.gz EFI

---

## Final State

- Secure Boot ON
- rEFInd shim-mediated
- Windows anti-cheat works
- Bazzite works
- Clean EFI

This setup survives updates and firmware changes.
