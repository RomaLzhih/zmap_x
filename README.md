Install: 
```{shell}
cmake -DCMAKE_INSTALL_PREFIX=../zmap_x/bin .
make -j4
make install

clear && make zmap && make install && sudo ../bin/sbin/zmap -p 80 -r 128 171.67.70.0/23
```
