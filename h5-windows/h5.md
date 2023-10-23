# h5 Windows

## a) Hello Window Salt! Tee Windowsille SLS-tiedostoon Salt-tila, joka tekee tiedoston nimeltä "suolaikkuna.txt".

Asensin [Salt-Minionin Windowsille](https://docs.saltproject.io/salt/install-guide/en/latest/topics/install-by-operating-system/windows.html). Sitten avasin Windowsin PowerShellin `Windows-näppäin+x` -> Windows PowerShell(Admin). Lataussivustolta sai scriptinpätkän, johon muutin omat halutut tietoni salttia varten:

```
msiexec /i Salt-Minion-3005.1-1-Py3-AMD64.msi.md5 /quiet /norestart MASTER=windowsmaster MINION_ID=windowsminion
```

Sitten testasin saltin toimivuutta yksinkertaisella komentorivikutsulla

```
PS C:\> salt-call --local cmd.run 'echo hei maailma'
local:
    hei maailma
```

Sitten loin C:\-asemalle temp-hakemiston, jonne loin suolaikkuna-hakemiston. Suolaikkuna-hakemistoon loin init.sls-tiedoston. Käytin näihin vanhasta tottumuksesta graafista käyttöliittymää. init.sls-sisältö ja tilan ajo:

```
C:/temp/suolaikkuna.txt:
  file.managed:
    - contents: "Hei Windows!"


PS C:\> salt-call --file-root=C:\temp\ --local state.apply suolaikkuna
local:
----------
          ID: C:/temp/suolaikkuna.txt
    Function: file.managed
      Result: True
     Comment: File C:/temp/suolaikkuna.txt updated
     Started: 20:59:30.047882
    Duration: 62.465 ms
     Changes:
              ----------
              diff:
                  New file

Summary for local
------------
Succeeded: 1 (changed=1)
Failed:    0
------------
Total states run:     1
Total run time:  62.465 ms

```

Sitten vielä testasin nopeasti tiedoston sisällön:

```
PS C:\> cat 'C:\temp\suolaikkuna.txt'
Hei Windows!
```

## b) Ei vihkoa, ei kynää. Kerää Windows-koneen tekniset tiedot tekstitiedostoon.

Avasin Windowsin PowerShellin taas Administrator-käyttöoikeuksilla. Loin tiedoston kirjoittamalla komentoriville

```
salt-call --local grains.items > windowstiedot.txt
```

Etsin tiedoston käsiini (Windows-näppäin -> kirjoitin windowstiedot.txt). Tiedostopolku oli hieman pelottavasti C:\system32\windowstiedot.txt. Tiedosto kuitenki sisälsi kaikki grains.itemsin antamat tiedot. System32 sisältää ymmärtääkseni Windowsin toiminnan kannalta kriittisiä tiedostoja, joten kopioin tiedoston vielä omaan kansioonsa toisaalle.

## c) Kop kop. Onko TCP-portti auki vai kiinni? Näytä esimerkit portin kokeilusta Linuxilla ja Windowsilla. Näytä kummallakin käyttöjärjestelmällä ainakin yksi avoin ja yksi suljettu portti. (Kokeile tätä vain omaan koneeseesi. Vieraiden koneiden ja verkkojen porttiskannaaminen on kiellettyä. Yksittäisen portin testaavat komennot ovat suositeltavia, esim. nc, tnc)

Asensin Ubuntulle netcatin

```
sudo apt-get install netcat
```

Sitten testasin Ubuntu-virtuaalikoneen portteja 

```
:~$ nc -vz 10.0.2.15 4505
Connection to 10.0.2.15 4505 port [tcp/*] succeeded!
:~$ nc -vz 10.0.2.15 4506
Connection to 10.0.2.15 4506 port [tcp/*] succeeded!
:~$ nc -vz 10.0.2.15 4507
nc: connect to 10.0.2.15 port 4507 (tcp) failed: Connection refused

```

Portit 4505 ja 4506 ovat auki salttia varten. Sitten windowsin testeihin:

```
PS C:\Windows\system32> tnc 10.0.2.15 -p 4505
WARNING: TCP connect to (10.0.2.15 : 4505) failed


ComputerName           : 10.0.2.15
RemoteAddress          : 10.0.2.15
RemotePort             : 4505
InterfaceAlias         : Ethernet
SourceAddress          : 10.0.2.15
PingSucceeded          : True
PingReplyDetails (RTT) : 0 ms
TcpTestSucceeded       : False


```

Kokeilin useilla eri porteilla (4506, 8888, 43), mutta tulos oli aina sama. Tämä saattaa johtua reitittimeni tulimuuriasetuksista, tai jostakin muusta asiasta. Ajanpuutteen vuoksi en kuitenkaan ehtinyt tutkia asiaa tämän tarkemmin.
