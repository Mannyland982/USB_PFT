# USB Partition & Format Tool

A lightweight, modular Windows utility designed to format, repartition, and recover USB flash drives. It specializes in setting up compliant **8GB FAT32 partitions** required by legacy systems, industrial CNC controllers, vintage synthesisers, and older car audio systems, while also providing one-click full-capacity restoration and unblocking of corrupted RAW drives.

![Platform](https://img.shields.io/badge/platform-Windows-0078D6)
![Python Version](https://img.shields.io/badge/python-3.9%2B-blue)
![License](https://img.shields.io/badge/license-MIT-green)

---

## ✨ Key Capabilities

- **Legacy & CNC Compatibility Mode (8GB FAT32):** Creates a fixed 8192 MB primary FAT32 partition. Essential for legacy hardware and machine tools that cannot recognize USB volumes larger than 8GB or non-FAT32 file systems.
- **Capacity Restoration:** Reclaims the full physical capacity of partitioned or reduced drives, formatting them into standard exFAT.
- **Corrupted / RAW Drive Unblocker:** Performs a low-level clean via Windows Diskpart to clear damaged partition tables, reset drive flags, and create a clean NTFS volume.
- **Hardware-Filtered Safety:** Uses WMI querying (`Win32_LogicalDisk` with `DriveType=2`) to scan and list **removable USB drives only**, helping prevent accidental modification of system or fixed hard disks.
- **Non-Blocking Asynchronous UI:** Operations run on isolated background daemon threads with animated visual feedback.
- **Bilingual Interface:** Instant switching between English and Italian at runtime.

---

## ⚠️ Important Requirements

- **Operating System:** Windows 10 or Windows 11.
- **Privileges:** **Must be executed as Administrator** (required for Windows `diskpart` and low-level volume manipulation).
- **Python:** Version 3.9 or newer.

---

## 🚀 Getting Started

### Installation

1. **Clone the repository:**
   ```bash
   git clone [https://github.com/Mannyland982/usb-partition-tool.git](https://github.com/Mannyland982/usb-partition-tool.git)
   cd usb-partition-tool
Run the tool:
Open a terminal/command prompt as Administrator and run:

Bash
python main.py
(Note: No third-party dependencies required. The tool relies entirely on Python's built-in standard library).
