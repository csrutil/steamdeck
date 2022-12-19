# Steam Deck

Here are some snippets that I did on my steam deck.

***⚠️⚠️⚠️I will not be responsible for any loss or damage***

## Telegram

Please join Telegram [Join Telegram](https://t.me/hackintash), if you have any questions

![steamdeck](https://i.imgur.com/2zdn2AI.png)

## Prepare

- Go to desk mode and set default password with `passwd`
- Enable R+W on the current running SteamOS by `steamos-readonly disable` - need sudo privilege
- Install common packages `pacman -S iperf3 tmux ethtool net-tools smemstat`

## SSH

I like to remote control the machine by SSH sometimes, so I need to enable the ssh, and please notice that you need to disable the password login and only allow use private key to login.

- Start ssh
  ```bash
  systemctl start sshd
  systemctl enable sshd

  # copy pub key to ~/.ssh/authorized_keys
  ssh-copy-id -i ~/.ssh/your_key_ed25519 deck@deck_ip
  ```

- Disable password login
  ```bash
  PasswordAuthentication no
  PubkeyAuthentication yes
  ```

## Journald logging

systemd-journald is a system service that collects and stores logging data, but we don't need those data probably. So I limit the log size to 50MB, you can change the size by your own.

  ```bash
  # the size for the log, 10 hours
  (A)(root@steamdeck ~)# du -sh /var/log/journal
  22M /var/log/journal
  ```

- Change the log size

  ```bash
  nano /etc/systemd/journald.conf

  [Journal]
  #Storage=auto
  #Compress=yes
  #Seal=yes
  #SplitMode=uid
  #SyncIntervalSec=5m
  #RateLimitIntervalSec=30s
  #RateLimitBurst=10000
  #SystemMaxUse=
  SystemMaxUse=50M
  ```

- Restart systemd-journald

  ```bash
  systemctl restart  systemd-journald
  ```

## Swap

Reduce the usage of swap file to extend the disk lifespan. the default value is 100, I changed it to 1.

> 1 Minimum amount of swapping without disabling it entirely (on kernels 3.5 and new, same as 0 on older kernels).

  ```bash
  (deck@steamdeck sysctl.d)$ cat /etc/sysctl.d/swappiness.conf
  vm.swappiness=100
  ```


## Spectre

From https://meltdownattack.com/

> Spectre breaks the isolation between different applications. It allows an attacker to trick error-free programs, which follow best practices, into leaking their secrets

We can check the CPU by `lscpu`, we can see `Spec store bypass/Spectre v1/Spectre v2` are mitigated. In my opinion, it's not necessary on the steam deck, and it will drop the performance a little bit(I can't sure how much). So I removed the patch.

![Spectre](https://i.imgur.com/g3r3Uzo.png)

You will get about ~10% performance(benchmark by 7zip)


## Donating 💸

Feel free to [Buy Me a Coffee](https://www.buymeacoffee.com/csrutil)

![Disabled Spectre](https://i.imgur.com/Svdm4Ul.png)

- Add `mitigations=off` to the GRUB_CMDLINE_LINUX_DEFAULT

  ```bash
  nano /etc/default/grub

  GRUB_CMDLINE_LINUX_DEFAULT="loglevel=3 quiet splash plymouth.ignore-serial-consoles module_blacklist=tpm amd_iommu=off amdgpu.gttsize=8128 spi_amd.speed_dev=1 audit=0 fbcon=vc:4-6 fbcon=rotate:1 mitigations=off"
  ```

- **Generate a new grub2 config file**

  ```bash
  update-grub
  ```

- 7zip Benchmark

  ```bash
  7-Zip [64] 17.04 : Copyright (c) 1999-2021 Igor Pavlov : 2017-08-28
  p7zip Version 17.04 (locale=en_US.UTF-8,Utf16=on,HugeFiles=on,64 bits,8 CPUs x64)

  x64
  CPU Freq: - - 64000000 - - - - - -

  RAM size:   14836 MB,  # CPU hardware threads:   8
  RAM usage:   1765 MB,  # Benchmark threads:      8

                         Compressing  |                  Decompressing
  Dict     Speed Usage    R/U Rating  |      Speed Usage    R/U Rating
           KiB/s     %   MIPS   MIPS  |      KiB/s     %   MIPS   MIPS

  22:      18214   721   2458  17720  |     268427   765   2994  22896
  23:      18601   741   2558  18952  |     261928   766   2959  22667
  24:      17542   746   2528  18861  |     256192   772   2912  22486
  25:      16701   757   2520  19069  |     250730   778   2867  22314
  ----------------------------------  | ------------------------------
  Avr:             741   2516  18651  |              770   2933  22590
  Tot:             756   2724  20621



  7-Zip [64] 17.04 : Copyright (c) 1999-2021 Igor Pavlov : 2017-08-28
  p7zip Version 17.04 (locale=en_US.UTF-8,Utf16=on,HugeFiles=on,64 bits,8 CPUs x64)

  x64
  CPU Freq: - - - - - - - - -

  RAM size:   14836 MB,  # CPU hardware threads:   8
  RAM usage:   1765 MB,  # Benchmark threads:      8

                         Compressing  |                  Decompressing
  Dict     Speed Usage    R/U Rating  |      Speed Usage    R/U Rating
           KiB/s     %   MIPS   MIPS  |      KiB/s     %   MIPS   MIPS

  22:      14703   612   2337  14304  |     227173   669   2897  19377
  23:      14520   621   2381  14795  |     236744   706   2902  20487
  24:      17248   737   2516  18546  |     254143   768   2904  22306
  25:      16457   747   2515  18791  |     247476   770   2859  22024
  ----------------------------------  | ------------------------------
  Avr:             679   2437  16609  |              728   2891  21049
  Tot:             704   2664  18829
  ```

## Snippets(REBOOT AFTER MODIFIED)

Journald log file size to 50MB

  ```bash
  sudo steamos-readonly disable

  cp /etc/systemd/journald.conf ~/Documents/
  cp /etc/default/grub ~/Documents/

  sudo sed -i '/SystemMaxUse/d' /etc/systemd/journald.conf
  sudo echo "SystemMaxUse=50M" >> /etc/systemd/journald.conf
  ```

Swap

  ```bash
  sudo sed -i 's/100/1/g' /etc/sysctl.d/swappiness.conf
  ```

Mitigations OFF

  ```bash
  sudo sed -i 's/GRUB_CMDLINE_LINUX_DEFAULT="[^"]*/& mitigations=off/' /etc/default/grub
  sudo update-grub
  ```

