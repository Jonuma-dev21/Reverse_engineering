(venv) jonuma@jonas-Bunt:~/Documents/Master1/S2/Cours/SDS/volatility3$ python vol.py -f ram.img linux.proc.Maps --pid 7295 --dump
Volatility 3 Framework 2.28.1
usage: vol.py [-h] [-c CONFIG] [--parallelism [{processes,threads,off}]] [-e EXTEND] [-p PLUGIN_DIRS] [-s SYMBOL_DIRS] [-v] [-l LOG] [-o OUTPUT_DIR]
              [-q] [-f FILE] [--write-config] [--save-config SAVE_CONFIG] [--clear-cache] [--cache-path CACHE_PATH] [--offline | -u URL]
              [--filters FILTERS] [--hide-columns [HIDE_COLUMNS ...]] [-r RENDERER] [--single-location SINGLE_LOCATION] [--stackers [STACKERS ...]]
              [--single-swap-locations [SINGLE_SWAP_LOCATIONS ...]]
              PLUGIN ...
vol.py: error: File does not exist: /home/jonuma/Documents/Master1/S2/Cours/SDS/volatility3/ram.img
(venv) jonuma@jonas-Bunt:~/Documents/Master1/S2/Cours/SDS/volatility3$ python vol.py -f ./../memoire/ram.img linux.proc.Maps --pid 7295 --dump
Volatility 3 Framework 2.28.1
Progress:  100.00		Stacking attempts finished            
PID	Process	Start	End	Flags	PgOff	Major	Minor	Inode	File Path	File output

7295	/bin/systemd	0x400000	0x410000	r-x	0x0	8	3	263912	/home/host/Downloads/bin/systemd	pid.7295.vma.0x400000-0x410000.dmp
7295	/bin/systemd	0x50f000	0x511000	rw-	0xf000	8	3	263912	/home/host/Downloads/bin/systemd	pid.7295.vma.0x50f000-0x511000.dmp
7295	/bin/systemd	0x511000	0x513000	rw-	0x0	0	0	0	Anonymous Mapping	pid.7295.vma.0x511000-0x513000.dmp
7295	/bin/systemd	0x11df000	0x11e0000	rw-	0x0	0	0	0	[heap]	pid.7295.vma.0x11df000-0x11e0000.dmp
7295	/bin/systemd	0x7ffc628b9000	0x7ffc628da000	rw-	0x0	0	0	0	[stack]	pid.7295.vma.0x7ffc628b9000-0x7ffc628da000.dmp
7295	/bin/systemd	0x7ffc62934000	0x7ffc62938000	r--	0x0	0	0	0	Anonymous Mapping	pid.7295.vma.0x7ffc62934000-0x7ffc62938000.dmp
7295	/bin/systemd	0x7ffc62938000	0x7ffc6293a000	r-x	0x0	0	0	0	[vdso]	pid.7295.vma.0x7ffc62938000-0x7ffc6293a000.dmp
(venv) jonuma@jonas-Bunt:~/Documents/Master1/S2/Cours/SDS/volatility3$ ls
API_CHANGES.md   LICENSE.txt                           pid.7295.vma.0x511000-0x513000.dmp              README.md    volshell.py
CITATION.cff     MANIFEST.in                           pid.7295.vma.0x7ffc628b9000-0x7ffc628da000.dmp  test         volshell.spec
CODING_STYLE.md  pid.7295.vma.0x11df000-0x11e0000.dmp  pid.7295.vma.0x7ffc62934000-0x7ffc62938000.dmp  venv         vol.spec
development      pid.7295.vma.0x400000-0x410000.dmp    pid.7295.vma.0x7ffc62938000-0x7ffc6293a000.dmp  volatility3
doc              pid.7295.vma.0x50f000-0x511000.dmp    pyproject.toml                                  vol.py
(venv) jonuma@jonas-Bunt:~/Documents/Master1/S2/Cours/SDS/volatility3$ file pid.7295.vma.0x400000-0x410000.dmp
pid.7295.vma.0x400000-0x410000.dmp: ELF 64-bit LSB executable, x86-64, version 1 (SYSV), statically linked, missing section headers at 66368
(venv) jonuma@jonas-Bunt:~/Documents/Master1/S2/Cours/SDS/volatility3$ sha256sum pid.7295.vma.0x400000-0x410000.dmp
4ee71841b003b87f70942cf40c054cb4b9f22a4e37ce0c66a19137930d23682d  pid.7295.vma.0x400000-0x410000.dmp
(venv) jonuma@jonas-Bunt:~/Documents/Master1/S2/Cours/SDS/volatility3$ cat \
pid.7295.vma.0x400000-0x410000.dmp \
pid.7295.vma.0x50f000-0x511000.dmp \
pid.7295.vma.0x511000-0x513000.dmp \
> reconstructed.elf
(venv) jonuma@jonas-Bunt:~/Documents/Master1/S2/Cours/SDS/volatility3$ sha256sum reconstructed.elf
7b6297662ce20d3837c6d2515354151a64cb727cb0c66ce11349e4802ee8914b  reconstructed.elf
(venv) jonuma@jonas-Bunt:~/Documents/Master1/S2/Cours/SDS/volatility3$ cat pid.7295.vma.0x400000-0x410000.dmp pid.7295.vma.0x50f000-0x511000.dmp > reconstructed.elf
(venv) jonuma@jonas-Bunt:~/Documents/Master1/S2/Cours/SDS/volatility3$ sha256sum reconstructed.elf
12661c9b284e6993e43a7dad7a987008da86f582bc6d4958e24ad39acef644dd  reconstructed.elf
