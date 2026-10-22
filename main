"""
USB Partition & Format Tool
A robust, modular desktop utility for Windows to configure USB flash drives 
with specific partition constraints (e.g., 8GB FAT32 for legacy/CNC systems), 
restore factory capacity, and recover corrupted/RAW storage units.
"""

from abc import ABC, abstractmethod
import ctypes
import os
import subprocess
import sys
import tempfile
import threading
import tkinter as tk
from tkinter import messagebox, ttk


# =============================================================================
# 1. LOCALIZATION DICTIONARY (IT / EN)
# =============================================================================

LANG = {
    "IT": {
        "title": "USB PARTITION & FORMAT TOOL",
        "subtitle": "Seleziona l'unità USB (Dispositivi Rimovibili):",
        "refresh": "Aggiorna unità",
        "btn_prepare": "⚡ CONFIGURA UNITÀ (8GB FAT32)",
        "btn_restore": "🔄 RIPRISTINA INTERA CAPIENZA",
        "btn_unblock": "🛠️ SBLOCCA E RIPARA UNITÀ CORROTTA",
        "disclaimer": (
            "⚠️ AVVISO IMPORTANTE: L'utilizzo delle funzioni di configurazione,\n"
            "ripristino o sblocco comporta la cancellazione definitiva di tutti i\n"
            "dati presenti sull'unità selezionata. Verificare con attenzione."
        ),
        "status_ready": "Pronto",
        "err_no_drive_title": "Errore",
        "err_no_drive_msg": "Seleziona prima una lettera di unità.",
        "conf_prep_title": "Conferma Configurazione",
        "conf_prep_msg": (
            "ATTENZIONE: Tutti i dati sull’unità {drive}: verranno cancellati\n"
            "per ridurla a 8GB in FAT32.\n\nVuoi procedere?"
        ),
        "conf_rest_title": "Conferma Ripristino",
        "conf_rest_msg": (
            "ATTENZIONE: L’unità {drive}: verrà completamente formattata\n"
            "e ripristinata alla capienza massima originaria.\n\nVuoi procedere?"
        ),
        "conf_unb_title": "Conferma Riparazione Forzata",
        "conf_unb_msg": (
            "ATTENZIONE: Questo comando ripulirà a fondo e sbloccherà l'unità {drive}:\n"
            "ripristinando la partizione in caso di errori o blocchi RAW.\n\nVuoi procedere?"
        ),
        "load_prep": "Partizionamento unità {drive}: in corso...",
        "load_rest": "Ripristino unità {drive}: in corso...",
        "load_unb": "Riparazione unità {drive}: in corso...",
        "success_title": "Operazione Riuscita",
        "footer_text": "Open Source Tool • Licenza MIT",
        "admin_err_title": "Accesso Negato",
        "admin_err_msg": (
            "Questo programma richiede i privilegi di Amministratore.\n"
            "Riavvia l'applicazione come Amministratore."
        ),
    },
    "EN": {
        "title": "USB PARTITION & FORMAT TOOL",
        "subtitle": "Select USB Flash Drive (Removable Devices):",
        "refresh": "Refresh drives",
        "btn_prepare": "⚡ CONFIGURE DRIVE (8GB FAT32)",
        "btn_restore": "🔄 RESTORE FULL CAPACITY",
        "btn_unblock": "🛠️ UNBLOCK & REPAIR CORRUPTED DRIVE",
        "disclaimer": (
            "⚠️ IMPORTANT DISCLAIMER: Using configuration, restoration,\n"
            "or unblock functions will permanently erase all data on the\n"
            "selected drive. Please verify carefully."
        ),
        "status_ready": "Ready",
        "err_no_drive_title": "Error",
        "err_no_drive_msg": "Please select a drive letter first.",
        "conf_prep_title": "Confirm Configuration",
        "conf_prep_msg": (
            "WARNING: All data on drive {drive}: will be erased\n"
            "to configure an 8GB FAT32 partition.\n\nDo you want to proceed?"
        ),
        "conf_rest_title": "Confirm Restoration",
        "conf_rest_msg": (
            "WARNING: Drive {drive}: will be fully formatted\n"
            "and restored to its original maximum capacity.\n\nDo you want to proceed?"
        ),
        "conf_unb_title": "Confirm Forced Repair",
        "conf_unb_msg": (
            "WARNING: This command will perform a low-level clean and unlock drive {drive}:\n"
            "rebuilding the partition table in case of RAW errors.\n\nDo you want to proceed?"
        ),
        "load_prep": "Partitioning drive {drive}: in progress...",
        "load_rest": "Restoring drive {drive}: in progress...",
        "load_unb": "Repairing drive {drive}: in progress...",
        "success_title": "Operation Successful",
        "footer_text": "Open Source Utility • MIT License",
        "admin_err_title": "Access Denied",
        "admin_err_msg": (
            "This application requires Administrator privileges.\n"
            "Please restart the application as Administrator."
        ),
    },
}


# =============================================================================
# 2. INTERFACES & ABSTRACTIONS (ISP & OCP)
# =============================================================================

class IAdminChecker(ABC):
    @abstractmethod
    def is_admin(self) -> bool:
        pass


class IDiskProvider(ABC):
    @abstractmethod
    def get_available_drives(self) -> list:
        pass


class IDiskPreparer(ABC):
    @abstractmethod
    def prepare_drive(self, drive_letter: str) -> tuple[bool, str]:
        pass


class IDiskRestorer(ABC):
    @abstractmethod
    def restore_drive(self, drive_letter: str) -> tuple[bool, str]:
        pass


class IDiskUnblocker(ABC):
    @abstractmethod
    def unblock_drive(self, drive_letter: str) -> tuple[bool, str]:
        pass


# =============================================================================
# 3. CONCRETE IMPLEMENTATIONS (SRP)
# =============================================================================

class WindowsAdminChecker(IAdminChecker):
    """Verifies whether the current process holds Windows Administrator rights."""

    def is_admin(self) -> bool:
        try:
            return ctypes.windll.shell32.IsUserAnAdmin()
        except Exception:
            return False


class WindowsUSBDiskProvider(IDiskProvider):
    """Scans and returns ONLY removable USB storage device letters (DriveType=2)."""

    def get_available_drives(self) -> list:
        usb_drives = []
        ps_command = (
            "Get-WmiObject Win32_LogicalDisk | Where-Object {$_.DriveType -eq 2} |"
            " Select-Object -ExpandProperty DeviceID"
        )

        try:
            startupinfo = subprocess.STARTUPINFO()
            startupinfo.dwFlags |= subprocess.STARTF_USESHOWWINDOW

            res = subprocess.run(
                ["powershell", "-Command", ps_command],
                capture_output=True,
                text=True,
                check=True,
                creationflags=subprocess.CREATE_NO_WINDOW,
                startupinfo=startupinfo,
            )
            lines = res.stdout.strip().splitlines()
            for line in lines:
                clean = line.replace(":", "").strip().upper()
                if clean and len(clean) == 1:
                    usb_drives.append(clean)
        except Exception:
            pass

        return usb_drives


class WindowsDiskpartPreparer(IDiskPreparer):
    """Uses Windows Diskpart to repartition the drive to an 8GB FAT32 volume."""

    def prepare_drive(self, drive_letter: str) -> tuple[bool, str]:
        clean_letter = drive_letter.replace(":", "").strip().upper()
        find_disk_cmd = (
            f"Get-Partition -DriveLetter {clean_letter} | Select-Object -ExpandProperty DiskNumber"
        )

        startupinfo = subprocess.STARTUPINFO()
        startupinfo.dwFlags |= subprocess.STARTF_USESHOWWINDOW

        try:
            res = subprocess.run(
                ["powershell", "-Command", find_disk_cmd],
                capture_output=True,
                text=True,
                check=True,
                creationflags=subprocess.CREATE_NO_WINDOW,
                startupinfo=startupinfo,
            )
            disk_num = res.stdout.strip()
            if not disk_num.isdigit():
                return False, "Failed to identify physical disk number."
        except subprocess.CalledProcessError:
            return False, "Error querying physical disk."

        diskpart_script = f"""select disk {disk_num}
clean
create partition primary size=8192
format fs=fat32 quick
assign letter={clean_letter}
exit
"""
        with tempfile.NamedTemporaryFile(delete=False, suffix=".txt", mode="w", encoding="ascii") as f:
            f.write(diskpart_script)
            temp_script_path = f.name

        try:
            subprocess.run(
                ["diskpart", "/s", temp_script_path],
                capture_output=True,
                text=True,
                check=True,
                creationflags=subprocess.CREATE_NO_WINDOW,
                startupinfo=startupinfo,
            )
            return True, f"Drive {clean_letter}: successfully configured to 8GB FAT32."
        except subprocess.CalledProcessError as e:
            error_msg = e.stderr.strip() if e.stderr else "Unknown error."
            return False, f"Partition operation failed. Details: {error_msg}"
        finally:
            if os.path.exists(temp_script_path):
                os.remove(temp_script_path)


class WindowsDiskpartRestorer(IDiskRestorer):
    """Restores the USB drive to its full factory capacity formatted with exFAT."""

    def restore_drive(self, drive_letter: str) -> tuple[bool, str]:
        clean_letter = drive_letter.replace(":", "").strip().upper()
        find_disk_cmd = (
            f"Get-Partition -DriveLetter {clean_letter} | Select-Object -ExpandProperty DiskNumber"
        )

        startupinfo = subprocess.STARTUPINFO()
        startupinfo.dwFlags |= subprocess.STARTF_USESHOWWINDOW

        try:
            res = subprocess.run(
                ["powershell", "-Command", find_disk_cmd],
                capture_output=True,
                text=True,
                check=True,
                creationflags=subprocess.CREATE_NO_WINDOW,
                startupinfo=startupinfo,
            )
            disk_num = res.stdout.strip()
            if not disk_num.isdigit():
                return False, "Failed to identify physical disk number."
        except subprocess.CalledProcessError:
            return False, "Error querying physical disk."

        diskpart_script = f"""select disk {disk_num}
clean
create partition primary
format fs=exfat quick
assign letter={clean_letter}
exit
"""
        with tempfile.NamedTemporaryFile(delete=False, suffix=".txt", mode="w", encoding="ascii") as f:
            f.write(diskpart_script)
            temp_script_path = f.name

        try:
            subprocess.run(
                ["diskpart", "/s", temp_script_path],
                capture_output=True,
                text=True,
                check=True,
                creationflags=subprocess.CREATE_NO_WINDOW,
                startupinfo=startupinfo,
            )
            return True, f"Drive {clean_letter}: restored to full original capacity (exFAT)."
        except subprocess.CalledProcessError as e:
            error_msg = e.stderr.strip() if e.stderr else "Unknown error."
            return False, f"Restore failed. Details: {error_msg}"
        finally:
            if os.path.exists(temp_script_path):
                os.remove(temp_script_path)


class WindowsDiskpartUnblocker(IDiskUnblocker):
    """Performs deep cleaning and partition reconstruction for RAW or corrupted drives."""

    def unblock_drive(self, drive_letter: str) -> tuple[bool, str]:
        clean_letter = drive_letter.replace(":", "").strip().upper()
        find_disk_cmd = (
            f"Get-Partition -DriveLetter {clean_letter} | Select-Object -ExpandProperty DiskNumber"
        )
        disk_num = ""

        startupinfo = subprocess.STARTUPINFO()
        startupinfo.dwFlags |= subprocess.STARTF_USESHOWWINDOW

        try:
            res = subprocess.run(
                ["powershell", "-Command", find_disk_cmd],
                capture_output=True,
                text=True,
                check=True,
                creationflags=subprocess.CREATE_NO_WINDOW,
                startupinfo=startupinfo,
            )
            disk_num = res.stdout.strip()
        except Exception:
            pass

        if not disk_num.isdigit():
            try:
                ps_disk_num = (
                    "Get-Disk | Where-Object { $_.BusType -eq 'USB' } | Select-Object -ExpandProperty Number"
                )
                res_fb = subprocess.run(
                    ["powershell", "-Command", ps_disk_num],
                    capture_output=True,
                    text=True,
                    check=True,
                    creationflags=subprocess.CREATE_NO_WINDOW,
                    startupinfo=startupinfo,
                )
                lines = res_fb.stdout.strip().splitlines()
                if lines:
                    disk_num = lines[0].strip()
            except Exception:
                pass

        if not disk_num.isdigit():
            return False, "Could not locate matching USB disk. Ensure the drive is plugged in."

        diskpart_script = f"""select disk {disk_num}
clean
create partition primary
format fs=ntfs quick
assign letter={clean_letter}
exit
"""
        with tempfile.NamedTemporaryFile(delete=False, suffix=".txt", mode="w", encoding="ascii") as f:
            f.write(diskpart_script)
            temp_script_path = f.name

        try:
            subprocess.run(
                ["diskpart", "/s", temp_script_path],
                capture_output=True,
                text=True,
                check=True,
                creationflags=subprocess.CREATE_NO_WINDOW,
                startupinfo=startupinfo,
            )
            return True, f"Drive {clean_letter}: unblocked, cleaned, and initialized successfully."
        except subprocess.CalledProcessError as e:
            error_msg = e.stderr.strip() if e.stderr else "Unknown error."
            return False, f"Failed to unblock disk. Details: {error_msg}"
        finally:
            if os.path.exists(temp_script_path):
                os.remove(temp_script_path)


# =============================================================================
# 4. CUSTOM WIDGET: ANIMATED LOADING RING
# =============================================================================

class CircularProgressRing(tk.Canvas):
    """Custom canvas-based smooth spinner for operation feedback."""

    def __init__(self, master, size=70, **kwargs):
        super().__init__(
            master,
            width=size,
            height=size,
            bg=kwargs.get("bg", "#1e1e1e"),
            highlightthickness=0,
        )
        self.size = size
        self.angle = 0
        self.is_running = False
        self.arc_item = None
        self._create_base()

    def _create_base(self):
        margin = 8
        self.bbox = (margin, margin, self.size - margin, self.size - margin)
        self.create_oval(self.bbox, outline="#333333", width=5)
        self.arc_item = self.create_arc(
            self.bbox,
            start=0,
            extent=90,
            outline="#2ea043",
            width=5,
            style="arc",
        )

    def start(self):
        self.is_running = True
        self._animate()

    def stop(self):
        self.is_running = False

    def _animate(self):
        if self.is_running:
            self.angle = (self.angle + 15) % 360
            self.itemconfig(self.arc_item, start=self.angle)
            self.after(40, self._animate)


# =============================================================================
# 5. USER INTERFACE (DESKTOP GUI)
# =============================================================================

class ApplicationUI:
    def __init__(
        self,
        root: tk.Tk,
        disk_provider: IDiskProvider,
        disk_preparer: IDiskPreparer,
        disk_restorer: IDiskRestorer,
        disk_unblocker: IDiskUnblocker,
    ):
        self.root = root
        self._disk_provider = disk_provider
        self._disk_preparer = disk_preparer
        self._disk_restorer = disk_restorer
        self._disk_unblocker = disk_unblocker

        self.current_lang = "EN"

        # Dark theme palette
        self.bg_color = "#121212"
        self.card_color = "#1e1e1e"
        self.fg_color = "#e0e0e0"
        self.accent_green = "#2ea043"
        self.accent_blue = "#1f6feb"
        self.accent_orange = "#d29922"
        self.btn_gray = "#333333"

        self.root.title("USB Partition & Format Tool")
        self.root.geometry("440x550")
        self.root.resizable(False, False)
        self.root.configure(bg=self.bg_color)

        self._apply_dark_title_bar()
        self._create_widgets()
        self._refresh_drives()

    def _apply_dark_title_bar(self):
        try:
            self.root.update()
            hwnd = ctypes.windll.user32.GetParent(self.root.winfo_id())
            for attr in (20, 19):
                val = ctypes.c_int(1)
                if (
                    ctypes.windll.dwmapi.DwmSetWindowAttribute(
                        hwnd, attr, ctypes.byref(val), ctypes.sizeof(val)
                    )
                    == 0
                ):
                    break
        except Exception:
            pass

    def _set_language(self, lang: str):
        self.current_lang = lang
        t = LANG[self.current_lang]

        self.lbl_title.config(text=t["title"])
        self.lbl_sub.config(text=t["subtitle"])
        self.btn_refresh.config(text=t["refresh"])
        self.btn_prepare.config(text=t["btn_prepare"])
        self.btn_restore.config(text=t["btn_restore"])
        self.btn_unblock.config(text=t["btn_unblock"])
        self.lbl_disclaimer.config(text=t["disclaimer"])
        self.lbl_status.config(text=t["status_ready"])
        self.lbl_footer.config(text=t["footer_text"])

        if lang == "IT":
            self.btn_it.config(bg="#1f6feb", fg="#ffffff")
            self.btn_en.config(bg="#222222", fg="#aaaaaa")
        else:
            self.btn_en.config(bg="#1f6feb", fg="#ffffff")
            self.btn_it.config(bg="#222222", fg="#aaaaaa")

    def _create_widgets(self):
        t = LANG[self.current_lang]

        header_frame = tk.Frame(self.root, bg=self.bg_color)
        header_frame.pack(pady=(12, 5), fill="x", padx=20)

        lang_frame = tk.Frame(header_frame, bg=self.bg_color)
        lang_frame.pack(side="right")

        self.btn_it = tk.Button(
            lang_frame,
            text="ITA",
            command=lambda: self._set_language("IT"),
            bg="#222222",
            fg="#aaaaaa",
            font=("Segoe UI", 8, "bold"),
            relief="flat",
            cursor="hand2",
            width=4,
        )
        self.btn_it.pack(side="left", padx=2)

        self.btn_en = tk.Button(
            lang_frame,
            text="ENG",
            command=lambda: self._set_language("EN"),
            bg="#1f6feb",
            fg="#ffffff",
            font=("Segoe UI", 8, "bold"),
            relief="flat",
            cursor="hand2",
            width=4,
        )
        self.btn_en.pack(side="left", padx=2)

        self.lbl_title = tk.Label(
            self.root,
            text=t["title"],
            font=("Segoe UI", 12, "bold"),
            bg=self.bg_color,
            fg="#ffffff",
        )
        self.lbl_title.pack(pady=(2, 2))

        self.lbl_sub = tk.Label(
            self.root,
            text=t["subtitle"],
            font=("Segoe UI", 9),
            bg=self.bg_color,
            fg="#aaaaaa",
        )
        self.lbl_sub.pack(pady=2)

        frame_controls = tk.Frame(self.root, bg=self.card_color, bd=0, highlightthickness=0)
        frame_controls.pack(pady=10, padx=20, fill="x", ipady=10)

        self.drive_var = tk.StringVar()
        self.combo_drives = ttk.Combobox(
            frame_controls,
            textvariable=self.drive_var,
            state="readonly",
            width=12,
            font=("Segoe UI", 11),
        )
        self.combo_drives.pack(pady=(10, 5))

        self.btn_refresh = tk.Button(
            frame_controls,
            text=t["refresh"],
            command=self._refresh_drives,
            bg=self.btn_gray,
            fg=self.fg_color,
            activebackground="#444444",
            activeforeground="#ffffff",
            font=("Segoe UI", 9),
            relief="flat",
            cursor="hand2",
            width=18,
        )
        self.btn_refresh.pack(pady=5)

        self.btn_prepare = tk.Button(
            self.root,
            text=t["btn_prepare"],
            command=self._on_prepare_click,
            bg=self.accent_green,
            fg="#ffffff",
            activebackground="#3fb950",
            activeforeground="#ffffff",
            font=("Segoe UI", 10, "bold"),
            relief="flat",
            cursor="hand2",
            width=32,
        )
        self.btn_prepare.pack(pady=(10, 5), ipady=6)

        self.btn_restore = tk.Button(
            self.root,
            text=t["btn_restore"],
            command=self._on_restore_click,
            bg=self.accent_blue,
            fg="#ffffff",
            activebackground="#388bfd",
            activeforeground="#ffffff",
            font=("Segoe UI", 10, "bold"),
            relief="flat",
            cursor="hand2",
            width=32,
        )
        self.btn_restore.pack(pady=5, ipady=6)

        self.btn_unblock = tk.Button(
            self.root,
            text=t["btn_unblock"],
            command=self._on_unblock_click,
            bg=self.accent_orange,
            fg="#ffffff",
            activebackground="#e3b341",
            activeforeground="#ffffff",
            font=("Segoe UI", 10, "bold"),
            relief="flat",
            cursor="hand2",
            width=32,
        )
        self.btn_unblock.pack(pady=5, ipady=6)

        self.lbl_disclaimer = tk.Label(
            self.root,
            text=t["disclaimer"],
            font=("Segoe UI", 8),
            bg=self.bg_color,
            fg="#888888",
            justify="center",
        )
        self.lbl_disclaimer.pack(pady=(6, 4))

        self.bottom_frame = tk.Frame(self.root, bg=self.bg_color)
        self.bottom_frame.pack(pady=5, fill="x", padx=20)

        self.frame_loader = tk.Frame(self.bottom_frame, bg=self.bg_color)

        self.progress_ring = CircularProgressRing(self.frame_loader, size=32, bg=self.bg_color)
        self.progress_ring.pack(side="left", padx=(0, 10))

        self.lbl_status = tk.Label(
            self.frame_loader,
            text=t["status_ready"],
            font=("Segoe UI", 9),
            bg=self.bg_color,
            fg="#888888",
            anchor="w",
            width=22,
        )
        self.lbl_status.pack(side="left")

        self.lbl_footer = tk.Label(
            self.bottom_frame,
            text=t["footer_text"],
            font=("Segoe UI", 8),
            bg=self.bg_color,
            fg="#666666",
        )
        self.lbl_footer.pack(side="right", padx=5)

    def _set_loading(self, active: bool, text: str = ""):
        state = "disabled" if active else "normal"
        self.combo_drives.config(state="readonly" if not active else "disabled")
        self.btn_refresh.config(state=state)
        self.btn_prepare.config(state=state)
        self.btn_restore.config(state=state)
        self.btn_unblock.config(state=state)

        if active:
            self.frame_loader.pack(side="left", fill="x")
            self.lbl_status.config(text=text)
            self.progress_ring.start()
        else:
            self.progress_ring.stop()
            self.frame_loader.pack_forget()

    def _refresh_drives(self):
        drives = self._disk_provider.get_available_drives()
        self.combo_drives["values"] = drives
        if drives:
            self.combo_drives.current(0)
        else:
            self.combo_drives.set("")

    def _on_prepare_click(self):
        t = LANG[self.current_lang]
        drive = self.drive_var.get()
        if not drive:
            messagebox.showerror(t["err_no_drive_title"], t["err_no_drive_msg"])
            return

        confirmed = messagebox.askyesno(
            t["conf_prep_title"], t["conf_prep_msg"].format(drive=drive)
        )
        if not confirmed:
            return

        self._set_loading(True, t["load_prep"].format(drive=drive))

        def background_task():
            success, message = self._disk_preparer.prepare_drive(drive)
            self.root.after(0, lambda: self._handle_task_result(success, message))

        threading.Thread(target=background_task, daemon=True).start()

    def _on_restore_click(self):
        t = LANG[self.current_lang]
        drive = self.drive_var.get()
        if not drive:
            messagebox.showerror(t["err_no_drive_title"], t["err_no_drive_msg"])
            return

        confirmed = messagebox.askyesno(
            t["conf_rest_title"], t["conf_rest_msg"].format(drive=drive)
        )
        if not confirmed:
            return

        self._set_loading(True, t["load_rest"].format(drive=drive))

        def background_task():
            success, message = self._disk_restorer.restore_drive(drive)
            self.root.after(0, lambda: self._handle_task_result(success, message))

        threading.Thread(target=background_task, daemon=True).start()

    def _on_unblock_click(self):
        t = LANG[self.current_lang]
        drive = self.drive_var.get()
        if not drive:
            messagebox.showerror(t["err_no_drive_title"], t["err_no_drive_msg"])
            return

        confirmed = messagebox.askyesno(
            t["conf_unb_title"], t["conf_unb_msg"].format(drive=drive)
        )
        if not confirmed:
            return

        self._set_loading(True, t["load_unb"].format(drive=drive))

        def background_task():
            success, message = self._disk_unblocker.unblock_drive(drive)
            self.root.after(0, lambda: self._handle_task_result(success, message))

        threading.Thread(target=background_task, daemon=True).start()

    def _handle_task_result(self, success: bool, message: str):
        t = LANG[self.current_lang]
        self._set_loading(False)
        if success:
            messagebox.showinfo(t["success_title"], message)
            self._refresh_drives()
        else:
            messagebox.showerror(t["err_no_drive_title"], message)


# =============================================================================
# 6. ENTRY POINT & DEPENDENCY INJECTION
# =============================================================================

def main():
    admin_checker = WindowsAdminChecker()

    if not admin_checker.is_admin():
        root = tk.Tk()
        root.withdraw()
        messagebox.showerror(
            "Access Denied",
            "This application requires Administrator privileges.\n"
            "Questo programma richiede i privilegi di Amministratore.",
        )
        sys.exit(1)

    disk_provider = WindowsUSBDiskProvider()
    disk_preparer = WindowsDiskpartPreparer()
    disk_restorer = WindowsDiskpartRestorer()
    disk_unblocker = WindowsDiskpartUnblocker()

    root = tk.Tk()
    ApplicationUI(root, disk_provider, disk_preparer, disk_restorer, disk_unblocker)
    root.mainloop()


if __name__ == "__main__":
    main()
