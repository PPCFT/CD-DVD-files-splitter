import os
import shutil
from tqdm import tqdm 

# 4440MB powinno zmieścić się na DVD-R
MAX_SIZE_MB = 4440 
MAX_SIZE_BYTES = MAX_SIZE_MB * 1024 * 1024

def get_all_files_with_sizes(root_dir):
    """
    Zwraca listę (ścieżka do pliku, rozmiar w bajtach) wszystkich plików
    w katalogu root_dir, wraz z ścieżką względną.
    """
    files = []
    for dirpath, dirnames, filenames in os.walk(root_dir):
        for f in filenames:
            full_path = os.path.join(dirpath, f)
            size = os.path.getsize(full_path)
            rel_path = os.path.relpath(full_path, root_dir)
            files.append((rel_path, size))
    return files

def split_files_into_bins(files, max_bin_size):
    """
    Dzieli listę plików (rel_path, size) na "koszyki", z których
    każdy ma łączny rozmiar <= max_bin_size.
    Sortuje pliki malejąco wg rozmiaru i przydziela według algorytmu
    first-fit decreasing, by minimalizować liczbę koszyków.
    """
    files = sorted(files, key=lambda x: x[1], reverse=True)
    bins = []
    for f, size in files:
        placed = False
        for b in bins:
            if b['size'] + size <= max_bin_size:
                b['files'].append((f, size))
                b['size'] += size
                placed = True
                break
        if not placed:
            bins.append({'files': [(f, size)], 'size': size})
    return bins

def copy_files_to_bins(src_root, dst_root, bins):
    """
    Kopiuje pliki do katalogu dst_root, rozdzielając je na podkatalogi
    'Disk_1', 'Disk_2' itd., a dalej zagnieżdża całość w folderze nazwanym
    jak katalog źródłowy (bez ścieżki), zachowując strukturę katalogów plików.
    """
    import os
    from tqdm import tqdm
    import shutil

    # Pobierz tylko nazwę katalogu źródłowego
    src_root_name = os.path.basename(os.path.normpath(src_root))

    for i, b in enumerate(bins, 1):
        disk_folder = os.path.join(dst_root, f'Disk_{i}', src_root_name)
        print(f'Kopiuję pliki do {os.path.join(dst_root, f"Disk_{i}")}:')
        for rel_path, _ in tqdm(b['files'], desc=f'Disk_{i}', unit='plik'):
            src_path = os.path.join(src_root, rel_path)
            dst_path = os.path.join(disk_folder, rel_path)
            os.makedirs(os.path.dirname(dst_path), exist_ok=True)
            shutil.copy2(src_path, dst_path)
        print(f'Zgrano {len(b["files"])} plików o łącznym rozmiarze {b["size"] / (1024*1024):.2f} MB do {os.path.join(dst_root, f"Disk_{i}")}')


def main():
    src_dir = input("Podaj ścieżkę do katalogu źródłowego: ").strip()
    dst_dir = input("Podaj ścieżkę do katalogu docelowego: ").strip()

    # Sprawdzenie czy istnieje katalog źródłowy
    if not os.path.isdir(src_dir):
        print("Podany katalog źródłowy nie istnieje.")
        return
    
    # Pobranie listy plików z rozmiarami
    files = get_all_files_with_sizes(src_dir)
    if not files:
        print("Katalog źródłowy nie zawiera plików.")
        return

    # Podział plików na koszyki (płyty)
    bins = split_files_into_bins(files, MAX_SIZE_BYTES)

    print(f"Liczba utworzonych katalogów (płyt): {len(bins)}")

    # Kopiowanie plików do katalogów docelowych
    copy_files_to_bins(src_dir, dst_dir, bins)

if __name__ == "__main__":
    main()
