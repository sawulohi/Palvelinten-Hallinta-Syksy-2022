# h2 Package-File Service

Testaukset tehty VirtualBoxin kautta Ubuntu 22.04 LTS sekä Debian 11 (Bullseye)-distroilla.

## x) Lue ja tiivistä
[Karvinen 2018: Salt Quickstart - Salt Stack Master and Slave on Ubuntu Linux](https://terokarvinen.com/2018/salt-quickstart-salt-stack-master-and-slave-on-ubuntu-linux/)
 - Ohjeistus Salt Masterin ja Salt Minionin asennukseen sekä Minionin konfigurointiin
 - Masterin asennuskomennot ja oman masterin osoitteen selvitys:
 
		$ sudo apt-get update
		$ sudo apt-get -y install salt-master
		$ hostname -I
		10.0.2.15

 - Palomuurin ollessa käytössä portit 4505/tcp ja 4506/tcp tulee avata
 - Slaven asennus ja konfigurointi masteriin yhdistämistä varten:
 
	`$ sudo apt-get update
	$ sudo apt-get -y install salt-minion
	$ sudoedit /etc/salt/minion`

Laitetaan minion config-tiedostoon masterin osoite ja minionille haluttu nimi.

 	master: 10.0.2.15
	id: minion-1

Tallenna ja poistu editorista. Käynnistä vielä demoni uudelleen, jotta säädetyt asetukset tulevat voimaan:

	$ sudo systemctl restart salt-minion.service

 - Hyväksy uuden orjan avain *masterilla*:
 
	`$ sudo salt-key -A
	The following keys are going to be accepted:
	Unaccepted Keys:
	minion-1
	Proceed? [n/Y] Y
	Key for minion minion-1 accepted.`

Testaukset esimerkiksi (*masterilla*):

	$sudo salt '*' cmd.run 'whoami'
	minion-1:
	    root
 
[Salt official documentation: Salt Getting Started Guide-kirjasta luvut Understanding SaltStack ja SaltStack Fundamentals ja SaltStack Configuration Management: Functions](https://docs.saltproject.io/en/getstarted/)
#### Understanding SaltStack
 - Yleiskatsaus Saltin toimintaan ja toimintalogiikkaan ja -filosofiaan
 - Salt on rakennettu toimimaan kevyenä, skaalautuvana ja reaaliaikaisena hallinnointityökaluna kaikille käyttöjärjestelmille (Olettaen, että niissä toimii python. Ja jos ei, niin niihinkin laitteisiin on saatavilla proxy-minion jolla hallinnoinin saa toimimaan.)
 - Myös automatisointi mahdollistuu
 - Hallinnointia voidaan toteuttaaa "infrastructure as code"-lähestymistavalla. 
 - Salt on helposti muokattavissa omiin käyttötarpeisiin plug-inien kautta. Näillä voidaan esim. jakaa tiedostoja minioneille tai valtuuttaa tietyt minionit tekemään haluttuja operaatioita, tai konfiguroida packageihin haluttuja asioita
 - Esim. Linuxin kanssa on mahdollista käskeä eri distroilla toimivat järjestelmät sivuuttamaan niille sopimattomat moduulin osat (esim. Debianin apt vs RedHatin yum)
 - Yhteydet Saltilla toimivat siten, että minionit ottavat yhteyden masteriin. Oletusportteina toimivat 4505 ja 4506
 - Minionit valtuutetaan yhteyteen masterin kanssa salatun avaimen kautta. Kaikki liikenne toteutetaan salatulla yhteydellä
 - Minioneita voidaan hallinnoida tilojen (state) kautta (tai salt pillarin kautta). State-tiedostoihin voidaan koodata haluttuja asioita, kielenä toimii YAML
 - Saltin avulla voidaan myös kerätä tietoa kohdejärjestelmistä grainsin kautta. Näiden avulla on myös mahdollista jaotella järjestelmiin tehtäviä asioita (esim. asennetaan Windows-koneisiin x-ohjelmisto ja Linux koneisiin y)
 - Salt toimii pythonilla, mutta pythonin osaaminen ei ole käytön kannalta välttämätöntä. Sillä pystytään kuitenkin esim. muokkaamaan Saltin alijärjestelmien asetuksia tai toiminnallisuuksia

#### SaltStack Fundamentals
 - Saltin perusteet (asennus, komentojen ajo, konfiguraatioden määrittely, komentojen ja konfiguraatioiden ajaminen määriteltyihin järjestelmiin)
 - Demoympäristön ohjeet VirtualBoxille Vagrantin avustuksella
 - Top-tiedoston luonti jolla ohjataan komennot oikeille minioneille
 - Vastaavat ohjeet löytyvät kotitehtävieni h1-osiosta, tai em. Tero Karvisen Salt quickstart-ohjeesta

#### SaltStack Configuration Management (Functions)
 - Funktiot (functions) ovat tilojen (state) "verbejä", eli ne kertovat mitä *tehdään*
 - Yhtenä esimerkkifunktiona käytetään examples.sls-tilaan käytettävää pkg.installed-funktiota:
 
`	install vim:
	  pkg.installed:
	    - name: vim
	$sudo salt 'minion1' state.apply examples
`
 - Sisältää myös esimerkkejä mm.  pakettien poistamisesta (pkg.removed), hakemistojen luomisesta (file.directory) ja palvelujen käynnissäolemisen varmistamisesta (service.running)
 - Kertoo myös, että state.apply eli tilaa ajaessa Salt etsii kohdehakemistosta init.sls-tiedoston ja tekee sen mukaiset asetukset


## a) Demonin asetukset. Säädä jokin demoni (asenna+tee asetukset+testaa) package-file-service -rakenteella. Ensin käsin: muista tehdä ja raportoida asennus ensin käsin, vasta sitten automatisoiden. Jos osaat hyvin, voit tehdä jonkin eri asetuksen kuin tunnilla. Harjoitusta varten tulee siis tehdä alusta ja raportoida samalla.

Asensin ensin uudelle minionilleni ssh-paketin (package)

	$sudo apt-get install ssh -y

Testasin ssh:n toimivuutta ottamalla minioniin yhteyden master-koneellani (service) (osoitteen saa selville $hostname -I):

	$ssh kayttaja3@10.0.2.5
	kayttaja3@10.0.2.5's password:

Yhteys toimi, sillä komentoriviin tuli näkyviin salasanan syötön jälkeen kysely koneen sormenjäljen hyväksymisestä:

	The authenticity of host '10.0.2.5 (10.0.2.5)' can't be established.
	ED25519 key fingerprint is SHA256:rQWrcUnqNMRiS49HAitUQSy00vYzE99Q7KL2ZLeKlrw.
	This key is not known by any other names
	Are you sure you want to continue connecting (yes/no/[fingerprint])? y
	Please type 'yes', 'no' or the fingerprint: yes
	Warning: Permanently added '10.0.2.5' (ED25519) to the list of known hosts.

	Welcome to Ubuntu 22.04.1 LTS (GNU/Linux 5.15.0-52-generic x86_64)

	 * Documentation:  https://help.ubuntu.com
	 * Management:     https://landscape.canonical.com
	 * Support:        https://ubuntu.com/advantage

	0 updates can be applied immediately.


	The programs included with the Ubuntu system are free software;
	the exact distribution terms for each program are described in the
	individual files in /usr/share/doc/*/copyright.

	Ubuntu comes with ABSOLUTELY NO WARRANTY, to the extent permitted by
	applicable law.

	kayttaja3@kayttaja3-VirtualBox:~$ exit
	logout
	Connection to 10.0.2.5 closed.

Sitten säädin sen asetuksiin ssh:n käyttämään porttia 8888 (file):

	$sudoedit /etc/ssh/sshd_config
	Port 8888

Tallensin ja poistuin, sitten käynnistin demonin uudestaan jotta muutokset tulevat voimaan:

	$sudo systemctl restart ssh

Kokeilin ottaa uuteen minioniin yhteyttä uudestaan samalla komennolla:

	$ssh kayttaja3@10.0.2.5
	ssh: connect to host 10.0.2.5 port 22: Connection refused
	
ssh näyttää käyttävän oletuksena porttia 22, korjataan tilanne lisäämällä komennon parametreihin uusi portti:

	$ssh kayttaja3@10.0.2.5 -p 8888
	kayttaja3@10.0.2.5's password:
	
Saatiin sama aiemmin saatu tervetuloa-viesti, ja yhteys oli taas käytettävissä.

Sitten sama saltilla masterin kautta minionille. Sotketaan kuitenkin ensin ssh minionilta:

	$sudo apt-get purge ssh -y
	$sudoedit /etc/ssh/sshd_config
	lisätään tänne konffitiedostoon jotain tekstiä
	
Tallennetaan tiedosto ja poistutaan.

Seuraavaksi siirrytään master-koneelle luomaan halutunlainen Salt-tila:

	$sudo mkdir /srv/salt/ssh
	$sudo nano /srv/salt/ssh/init.sls
	ssh:
	  pkg.installed

	/etc/ssh/sshd_config:
  	file.managed:
	    - source: "salt://ssh/sshd_config"

	ssh.service:
	  service.running:
	    - watch:
	      - file: /etc/ssh/sshd_config

Tämän lisäksi luotuun hakemistoon tarvitaan myös tiedosto, jota state voi jakaa myös minioneille. Tämä saadaan vaikka masterilta kopioimalla:

	$sudo cp /etc/ssh/sshd_config /srv/salt/ssh/
	$sudoedit /srv/salt/ssh/srv/salt/ssh/sshd_config
	Port 8888

Tallensin, poistuin ja ajoin uuden salt-tilan:

	$sudo salt '*' state.apply ssh
	good_ubuntu:
	----------
	          ID: ssh
	    Function: pkg.installed
	      Result: True
	     Comment: The following packages were installed/updated: ssh
	     Started: 20:05:01.149444
	    Duration: 12146.383 ms
	     Changes:   
	              ----------
	              ssh:
	                  ----------
	                  new:
	                      1:8.9p1-3
	                  old:
	----------
	          ID: /etc/ssh/sshd_config
	    Function: file.managed
	      Result: True
	     Comment: File /etc/ssh/sshd_config updated
	     Started: 20:05:13.303993
	    Duration: 34.605 ms
	     Changes:   
	              ----------
	              diff:
	                  --- 
	                  +++ 
	                  @@ -1,4 +1,4 @@
	                  -lisätään tänne tiedostoon jotain tekstiäPort 8888
	                  +Port 8888
	                   # This is the sshd server system-wide configuration file.  See
	                   # sshd_config(5) for more information.
	                   
	----------
	          ID: ssh.service
	    Function: service.running
	      Result: True
	     Comment: Service restarted
	     Started: 20:05:13.368243
	    Duration: 919.229 ms
	     Changes:   
	              ----------
	              ssh.service:
	                  True
	
	Summary for good_ubuntu
	------------
	Succeeded: 3 (changed=3)
	Failed:    0
	------------
	Total states run:     3
	Total run time:  13.100 s
	
Staten ajaminen siis tapahtui onnistuneesti. Myös aiemmin tehty sotku konffitiedostoon näkyy tehdyissä muutoksissa.

b)

Tein tämän harjoituksen jo edellisellä viikolla (h1 tehtävissä). Tässä kertauksena:

Ohjeet saatu: [https://terokarvinen.com/2018/salt-quickstart-salt-stack-master-and-slave-on-ubuntu-linux/](https://terokarvinen.com/2018/salt-quickstart-salt-stack-master-and-slave-on-ubuntu-linux/)

	$ sudo apt-get update
  	$ sudo apt-get -y install salt-master
  	$ sudo apt-get -y install salt-minion
  
Tarkistin oman osoitteeni seuraavaksi:

 	$ hostname -I
  
Lisäsin nämä saltin configeihin:

	$ sudoedit /etc/salt/minion
  
Lisäsin seuraavat rivit, $HOSTNAME on hostname-komennosta saatu tieto:

	master: $HOSTNAME
	id: slave1
  
Seuraavaksi hyväksyin salt-avaimen:

	$ sudo salt-key -A
	The following keys are going to be accepted:
	Unaccepted Keys:
	slave1
	Proceed? [n/Y] Y
	Key for minion slave1 accepted.

Toiminnallisuuden testaaminen onnistui näppärästi a)-kohdassa tämän viikon harjoituksia, kun kokeilin ssh:n päivittämistä lähiverkon yli.

## c) Aja jokin tila paikallisesti ilman master-slave arkkitehtuuria. Analysoi debug-tulostetta. 'sudo salt-call --local state.apply hellotero -l debug'

Ajetaan edellisellä viikolla tehty hello-state:

	$sudo salt-call --local state.apply hello -l debug
	[DEBUG   ] Reading configuration from /etc/salt/minion
	[DEBUG   ] Including configuration from '/etc/salt/minion.d/_schedule.conf'
	[DEBUG   ] Reading configuration from /etc/salt/minion.d/_schedule.conf
	[DEBUG   ] Override  __grains__: <module 'salt.loaded.int.log_handlers.sentry_mod' from '/usr/lib/python3/dist-packages/salt/log/handlers/sentry_mod.py'>
	[DEBUG   ] Configuration file path: /etc/salt/minion
	[WARNING ] Insecure logging configuration detected! Sensitive data may be logged.
	[DEBUG   ] Grains refresh requested. Refreshing grains.
	[DEBUG   ] Reading configuration from /etc/salt/minion
	[DEBUG   ] Including configuration from '/etc/salt/minion.d/_schedule.conf'
	[DEBUG   ] Reading configuration from /etc/salt/minion.d/_schedule.conf
	[DEBUG   ] Override  __utils__: <module 'salt.loaded.int.grains.zfs' from '/usr/lib/python3/dist-packages/salt/grains/zfs.py'>
	[DEBUG   ] Elapsed time getting FQDNs: 0.03631401062011719 seconds
	[DEBUG   ] LazyLoaded zfs.is_supported
	[DEBUG   ] Determining pillar cache
	[DEBUG   ] LazyLoaded jinja.render
	[DEBUG   ] LazyLoaded yaml.render
	[DEBUG   ] LazyLoaded jinja.render
	[DEBUG   ] LazyLoaded yaml.render
	[DEBUG   ] LazyLoaded state.apply
	[DEBUG   ] LazyLoaded direct_call.execute
	[DEBUG   ] LazyLoaded saltutil.is_running
	[DEBUG   ] Override  __grains__: <module 'salt.loaded.int.module.grains' from '/usr/lib/python3/dist-packages/salt/modules/grains.py'>
	[DEBUG   ] LazyLoaded grains.get
	[DEBUG   ] LazyLoaded config.get
	[DEBUG   ] LazyLoaded roots.envs
	[DEBUG   ] Could not LazyLoad roots.init: 'roots.init' is not available.
	[DEBUG   ] Updating roots fileserver cache
	[DEBUG   ] Gathering pillar data for state run
	[DEBUG   ] Determining pillar cache
	[DEBUG   ] LazyLoaded jinja.render
	[DEBUG   ] LazyLoaded yaml.render
	[DEBUG   ] Finished gathering pillar data for state run
	[INFO    ] Loading fresh modules for state activity
	[DEBUG   ] LazyLoaded jinja.render
	[DEBUG   ] LazyLoaded yaml.render
	[DEBUG   ] Unable to list dir: /srv/salt/hello/sotkua
	[DEBUG   ] Unable to list dir: /srv/salt/hello/init.sls
	[DEBUG   ] Unable to list dir: /srv/salt/apache2/init.sls
	[DEBUG   ] Unable to list dir: /srv/salt/foo/init.sls
	[DEBUG   ] Unable to list dir: /srv/salt/ssh/init.sls
	[DEBUG   ] Unable to list dir: /srv/salt/ssh/sshd_config
	[DEBUG   ] Could not find file 'salt://hello.sls' in saltenv 'base'
	[DEBUG   ] In saltenv 'base', looking at rel_path 'hello/init.sls' to resolve 'salt://hello/init.sls'
	[DEBUG   ] In saltenv 'base', ** considering ** path '/var/cache/salt/minion/files/base/hello/init.sls' to resolve 'salt://hello/init.sls'
	[DEBUG   ] compile template: /var/cache/salt/minion/files/base/hello/init.sls
	[DEBUG   ] Jinja search path: ['/var/cache/salt/minion/files/base']
	[DEBUG   ] LazyLoaded roots.envs
	[DEBUG   ] Could not LazyLoad roots.init: 'roots.init' is not available.
	[PROFILE ] Time (in seconds) to render '/var/cache/salt/minion/files/base/hello/init.sls' using 'jinja' renderer: 0.030007600784301758
	[DEBUG   ] Rendered data from file: /var/cache/salt/minion/files/base/hello/init.sls:
	/tmp/hellojp123:
	  file.managed:
	    - contents: "Foo is so bar"
	/tmp/toka:
	  file.managed:
	    - contents: "bar is so foo"
	
	[DEBUG   ] Results of YAML rendering: 
	OrderedDict([('/tmp/hellojp123', OrderedDict([('file.managed', [OrderedDict([('contents', 'Foo is so bar')])])])), ('/tmp/toka', OrderedDict([('file.managed', [OrderedDict([('contents', 'bar is so foo')])])]))])
	[PROFILE ] Time (in seconds) to render '/var/cache/salt/minion/files/base/hello/init.sls' using 'yaml' renderer: 0.0003921985626220703
	[DEBUG   ] LazyLoaded config.option
	[DEBUG   ] LazyLoaded file.managed
	[INFO    ] Running state [/tmp/hellojp123] at time 20:28:04.735935
	[INFO    ] Executing state file.managed for [/tmp/hellojp123]
	[DEBUG   ] LazyLoaded file.source_list
	[INFO    ] File changed:
	New file
	[INFO    ] Completed state [/tmp/hellojp123] at time 20:28:04.744001 (duration_in_ms=8.065)
	[INFO    ] Running state [/tmp/toka] at time 20:28:04.744154
	[INFO    ] Executing state file.managed for [/tmp/toka]
	[INFO    ] File changed:
	New file
	[INFO    ] Completed state [/tmp/toka] at time 20:28:04.746816 (duration_in_ms=2.662)
	[DEBUG   ] File /var/cache/salt/minion/accumulator/139642061692768 does not exist, no need to cleanup
	[DEBUG   ] LazyLoaded state.check_result
	[DEBUG   ] LazyLoaded highstate.output
	[DEBUG   ] LazyLoaded nested.output
	[DEBUG   ] LazyLoaded nested.output
	local:
	----------
	          ID: /tmp/hellojp123
	    Function: file.managed
	      Result: True
	     Comment: File /tmp/hellojp123 updated
	     Started: 20:28:04.735936
	    Duration: 8.065 ms
	     Changes:   
	              ----------
	              diff:
	                  New file
	----------
	          ID: /tmp/toka
	    Function: file.managed
	      Result: True
	     Comment: File /tmp/toka updated
	     Started: 20:28:04.744154
	    Duration: 2.662 ms
	     Changes:   
	              ----------
	              diff:
	                  New file
	
	Summary for local
	------------
	Succeeded: 2 (changed=2)
	Failed:    0
	------------
	Total states run:     2
	Total run time:  10.727 ms 

Tulosteen loppuosa vaikuttaa tyypilliseltä salt.staten ajamiselta. Sitä edeltävässä osiossa on kuitenkin paljon rivejä [DEBUG]- tai [INFO]-tägeillä varustettuna.
	`[DEBUG   ] Reading configuration from /etc/salt/minion`
Tämä rivi esimerkiksi vaikuttaa kertovan, että se lukee konfiguraatiotietoja paikallisesta /etc/salt/minion -hakemistosta.

	[DEBUG   ] Rendered data from file: /var/cache/salt/minion/files/base/hello/init.sls:
	/tmp/hellojp123:
	  file.managed:
	    - contents: "Foo is so bar"
	/tmp/toka:
	  file.managed:
	    - contents: "bar is so foo"
Tässä näkyy selkeästi init.sls -tiedoston sisältö, vaikka lähtökansio on eri kuin alkuperäisen tiedoston sijainti. Salt ilmeisesti käsittelee tietoja cachen kautta, ei suoraan? Tämä voisi olla esimerkiksi sen takia, ettei alkuperäisiin tiedostoihin "koskettaisi" tiloja ajettaessa?

	[INFO    ] Running state [/tmp/hellojp123] at time 20:28:04.735935
	[INFO    ] Executing state file.managed for [/tmp/hellojp123]
	[INFO    ] Completed state [/tmp/toka] at time 20:28:04.746816 (duration_in_ms=2.662)
	
INFO-tägein varustellut rivit näyttävät olevan lokitietoja, kuten aikaleimoja tai tehtyjä operaatioita.


