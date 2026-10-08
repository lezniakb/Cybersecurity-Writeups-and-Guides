task 1.

find / -name "*" 2>/dev/null | grep -iE "8v2l|bny0|c4zx|d8b3|fhl1|oimo|pfbd|rmfx|srsq|uqyw|v2vb|x1uy" > file_locations.txt

while read names; do ls -hal $names; done < file_locations.txt

answer 1: D8B3 v2Vb

task 2.
[new-user@ip-10-112-140-253 files]$ pwd
/home/new-user/files

[new-user@ip-10-112-140-253 files]$ while read names; do cp -p $names .; done < ../file_locations.txt

find . -exec grep -E '[0-9]+\.[0-9]+\.[0-9]+\.[0-9]+' {} \; 2>/dev/null
we verified there is the IP now which file is it?
find . -exec grep -El '[0-9]+\.[0-9]+\.[0-9]+\.[0-9]+' {} \; 2>/dev/null
./oiMO

answer 2: oiMO

task 3.
for file in *; do sha1sum $file; done | grep -iE "9d54"
9d54da7584015647ba052173b84d45e8007eba94  c4ZX

answer 3: c4ZX

task 4.
Tricky question. 
for file in *; do wc -l $file; done
209 8V2L
209 c4ZX
209 D8B3
209 FHl1
209 oiMO
209 PFbD
209 rmfX
209 SRSq
209 uqyw
209 v2Vb
209 X1Uy

we can see no file with 230 lines. But there is one file listed by the cretor of the room and one that wasn't found on the machine: bny0. This must be it!

answer 4: bny0

task 5. 
 while read p; do ls -hal $p | stat -c '%n  ---  %u' $p; done < file_locations.txt 
/mnt/D8B3  ---  501
/mnt/c4ZX  ---  501
/var/FHl1  ---  501
/var/log/uqyw  ---  501
/opt/PFbD  ---  501
/opt/oiMO  ---  501
/media/rmfX  ---  501
/etc/8V2L  ---  501
/etc/ssh/SRSq  ---  501
/home/v2Vb  ---  501
/X1Uy  ---  502

Answer 5: X1Uy

task 6.
 while read p; do ls -hal $p | stat -c '%n  ---  %A' $p; done < file_locations.txt | grep "x"
/etc/8V2L  ---  -rwxrwxr-x

answer 6: 8V2L
