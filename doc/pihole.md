# Pi-hole

## Pi-hole: il DNS fatto in casa per proteggerci

Pi-hole è una soluzione open source che funge da server DNS locale con capacità di filtraggio, progettato principalmente per bloccare pubblicità e malware su reti domestiche o aziendali. Opera come un "filtro" tra gli utenti della rete e il mondo internet, intercettando le richieste DNS e confrontandole con liste nere di domini noti per contenere pubblicità invasiva, spyware, ransomware, malware, cryptoware, cryptominer e altri tipi di contenuti indesiderati. Installato su un dispositivo dedicato (come Raspberry Pi), può proteggere tutti i dispositivi connessi alla rete, bloccando automaticamente le richieste a domini dannosi o pubblicitari senza necessità di configurazione su ciascun dispositivo.

## Configurazione e personalizzazione di Pi-hole

Per configurare Pi-hole, è necessario installarlo su un dispositivo che fungerà da server DNS, come Raspberry Pi, utilizzando guide specifiche disponibili sul sito ufficiale di Pi-hole. Una volta installato, l'utente può accedere all’interfaccia web di Pi-hole per personalizzare le impostazioni, inclusa la gestione delle liste nere e whitelist, il monitoraggio del traffico DNS e le statistiche sulla rete.

Nel nostro caso verrà installato come container LXC in proxmox. 
E' installato utilizzando il seguente script: https://pimox-scripts.com/scripts?id=pihole
Conviene avere l'accortezza di installarlo con IP Statico (nel mio caso 192.168.1.1) per facilitarne la configurazione.

Si può accedere all'interfaccia di configurazione puntando all'indirizzo : http://192.168.1.1/admin

> _Impostazione della password_
>
> Occorre creare la password di accesso prima di poter accedere la prima volta:
> 
> da Proxmox accedere alla _Shell_ del container appena creato.
> Eseguire il comando : `pihole -a -p` e creare la password
>



### Pi-hole come DNS su tutti i dispositivi

Affinché il tutto sia funzionante occorre impostare in tutti i dispositivi l'indirizzo di Pi-Hole come server DNS.
Siccome questa operazione potrebbe essere lunga e, per alcuni dispositivi, potrebbe non essere facilmente fattibile
si sceglie la soluzione di _attivare il servizio DHCP_ in Pi-hole, così sarà questo servizio a programmare i
dispositivi con tutti gli indirizzi necessari.

1. disabilitare il servizio DHCP dal router o da eventuale altro server
2. Dal menù `Settings/DHCP` abilitare **DHCP server enabled**
3. **Range of IP adresses to hand out** : From: `192.168.1.50` To: `192.168.1.199`
4. **Router(gateway) IP address** : Router: `192.168.1.254`



<!-- 

Fare riferimento al sito:

https://www.navigaresenzapubblicita.org/progetto-pi-hole-il-dns-fatto-in-casa-per-proteggerci-da-domini-che-contengono-malware-spyware-ranswomware-cryptoware-cryptominer-e-bloccare-i-domini-siti-web-che-contengono-la-pubblicita/

https://www.navigaresenzapubblicita.org/configurazione-e-personalizzazione-di-pi-hole/


1) Liste da aggiungere per il bloccare siti web/server malevoli

Queste liste comprendono i siti web che effettuano phising, scam (come ad esempio siti web contraffatti/di truffa), oppure che contengono: malware, ransomware, virus o ancora IP di malintenzionati/hacker, e per finire siti di Referrer spam.

Nota: queste liste non bloccano la pubblicità e/o tracker, non causano “blocchi anomali” durante la navigazione o il durante gioco online e sono utili per tutti i dispositivi come : Pc, Console (Nintendo Switch, Playstation, Xbox), Smartphone, Smart Tv, etc. Consiglio di tenerle sempre attive per qualsiasi dispositivo e quindi lasciarle impostaste nel gruppo di default.

https://blocklistproject.github.io/Lists/ransomware.txt
https://phishing.army/download/phishing_army_blocklist_extended.txt
https://raw.githubusercontent.com/DandelionSprout/adfilt/master/Alternate%20versions%20Anti-Malware%20List/AntiMalwareDomains.txt
https://s3.amazonaws.com/lists.disconnect.me/simple_malvertising.txt
https://v.firebog.net/hosts/Prigent-Crypto.txt
https://raw.githubusercontent.com/migueldemoura/ublock-umatrix-rulesets/master/Hosts/malware
https://blocklistproject.github.io/Lists/scam.txt
https://www.blocklist.de/downloads/export-ips_all.txt
Nota: blocco degli ip che hanno o stanno attaccando i server, utile per difendersi anche per il traffico in entrata o se stiamo usando programmi p2p tipo: torrent, emule, ecc.
https://gitlab.com/quidsup/notrack-blocklists/raw/master/notrack-malware.txt
https://osint.digitalside.it/Threat-Intel/lists/latestdomains.txt
https://raw.githubusercontent.com/PolishFiltersTeam/KADhosts/master/KADhosts.txt
https://raw.githubusercontent.com/Spam404/lists/master/main-blacklist.txt
https://raw.githubusercontent.com/r-a-y/mobile-hosts/master/AdguardMobileSpyware.txt
https://raw.githubusercontent.com/mitchellkrogza/The-Big-List-of-Hacked-Malware-Web-Sites/master/hacked-domains.list
https://urlhaus.abuse.ch/downloads/hostfile/
https://raw.githubusercontent.com/FadeMind/hosts.extras/master/add.Risk/hosts
https://bitbucket.org/ethanr/dns-blacklists/raw/8575c9f96e5b4a1308f2f12394abd86d0927a4a0/bad_lists/Mandiant_APT1_Report_Appendix_D.txt
https://raw.githubusercontent.com/MajkiIT/polish-ads-filter/master/polish-pihole-filters/kad_host.txt
https://blocklistproject.github.io/Lists/phishing.txt
https://blocklistproject.github.io/Lists/fraud.txt
https://raw.githubusercontent.com/kboghdady/youTube_ads_4_pi-hole/master/crowed_list.txt
https://raw.githubusercontent.com/FadeMind/hosts.extras/master/add.Spam/hosts
https://gitlab.com/gerowen/old-malware-domains-ad-list/-/raw/master/malwaredomainslist.txt
https://raw.githubusercontent.com/firehol/blocklist-ipsets/master/firehol_level2.netset
https://raw.githubusercontent.com/scafroglia93/blocklists/master/blocklists-malware-traffic.txt
https://raw.githubusercontent.com/infinitytec/blocklists/master/scams-and-phishing.txt
https://raw.githubusercontent.com/durablenapkin/scamblocklist/master/hosts.txt
https://raw.githubusercontent.com/XionKzn/PiHole-Lists/master/PiHole/Archive/Cerber_Ransomware.txt
https://raw.githubusercontent.com/XionKzn/PiHole-Lists/master/PiHole/PiHole_HOSTS_Spyware_HOSTS.txt
https://raw.githubusercontent.com/DRSDavidSoft/additional-hosts/master/domains/blacklist/fake-domains.txt
http://blacklists.ntop.org/blacklist-hostnames.txt
https://raw.githubusercontent.com/Ultimate-Hosts-Blacklist/MalwareDomainList.com/master/domains.list
https://raw.githubusercontent.com/stamparm/blackbook/master/blackbook.txt
https://raw.githubusercontent.com/stamparm/aux/master/maltrail-malware-domains.txt
http://blacklists.ntop.org/blacklist-ip.txt
https://bl.isx.fr/raw
https://raw.githubusercontent.com/blocklistproject/Lists/master/malware.txt
https://www.botvrij.eu/data/ioclist.hostname.raw
https://cinsscore.com/list/ci-badguys.txt – Dopo vari giorni di test risulta funzionante e quindi viene ripristinata
https://danger.rulez.sk/projects/bruteforceblocker/blist.php
https://raw.githubusercontent.com/gnxsecurity/gnx-threat-intelligence/master/latest-blacklist.raw
https://blocklist.greensnow.co/greensnow.txt
https://raw.githubusercontent.com/matomo-org/referrer-spam-blacklist/master/spammers.txt
https://malware-filter.gitlab.io/malware-filter/urlhaus-filter-domains.txt (in caso di problemi usare il link mirror https://curbengh.github.io/urlhaus-filter/urlhaus-filter-domains.txt)
https://malware-filter.gitlab.io/malware-filter/phishing-filter-domains.txt (in caso di problemi usare il link mirror https://curbengh.github.io/phishing-filter/phishing-filter-domains.txt)
https://raw.githubusercontent.com/olbat/ut1-blacklists/master/blacklists/malware/domains
https://raw.githubusercontent.com/RPiList/specials/master/Blocklisten/malware
https://raw.githubusercontent.com/RPiList/specials/master/Blocklisten/crypto
https://raw.githubusercontent.com/iam-py-test/my_filters_001/main/Alternative%20list%20formats/antimalware_domains.txt
https://raw.githubusercontent.com/scafroglia93/blocklists/master/blocklists-main.txt
https://hole.cert.pl/domains/domains.txt
https://raw.githubusercontent.com/TheAntiSocialEngineer/AntiSocial-BlockList-UK-Community/main/UK-Community.txt
https://raw.githubusercontent.com/Te-k/stalkerware-indicators/master/generated/hosts
https://v.firebog.net/hosts/Prigent-Malware.txt
https://raw.githubusercontent.com/hagezi/dns-blocklists/main/adblock/fake.txt
https://raw.githubusercontent.com/hagezi/dns-blocklists/main/adblock/tif.txt
https://raw.githubusercontent.com/AssoEchap/stalkerware-indicators/master/generated/hosts
https://raw.githubusercontent.com/elliotwutingfeng/Inversion-DNSBL-Blocklists/main/Google_hostnames.txt
https://raw.githubusercontent.com/no-cmyk/Search-Engine-Spam-Domains-Blocklist/master/blocklist.txt
http://www.taz.net.au/Mail/SpamDomains
https://raw.githubusercontent.com/Th3M3/blocklists/master/malware.list
https://raw.githubusercontent.com/RPiList/specials/master/Blocklisten/spam.mails
https://raw.githubusercontent.com/RPiList/specials/master/Blocklisten/notserious
Nota: Questa lista blocca negozi falsi e altri tipi di fregature.
https://gitlab.com/intr0/iVOID.GitLab.io/raw/master/iVOID.hosts
https://raw.githubusercontent.com/RPiList/specials/master/Blocklisten/Phishing-Angriffe
https://raw.githubusercontent.com/AdroitAdorKhan/antipopads-re/master/formats/domains.txt
Nota: Questa lista blocca i pop-up malevoli che si aprono automaticamente (spesso sono creati in java script e riescono ad eludere anche il blocco presente sul browser).
https://raw.githubusercontent.com/durablenapkin/scamblocklist/master/hosts.txt
https://raw.githubusercontent.com/soteria-nou/domain-list/master/fake.txt
https://raw.githubusercontent.com/mitchellkrogza/phishing/main/add-domain
https://malware-filter.gitlab.io/malware-filter/vn-badsite-filter-domains.txt
https://raw.githubusercontent.com/desbma/referer-spam-domains-blacklist/master/spammers.txt
https://raw.githubusercontent.com/scafroglia93/blocklists/master/blocklists-eset.txt
https://raw.githubusercontent.com/scafroglia93/blocklists/master/blocklists-microsoft.txt
https://raw.githubusercontent.com/DevSpen/scam-links/master/src/links.txt
https://raw.githubusercontent.com/stonecrusher/filterlists-pihole/master/watchlist-internet-ph.txt
https://raw.githubusercontent.com/bongochong/CombinedPrivacyBlockLists/master/NoFormatting/cpbl-ctld.txt
https://zonefiles.io/f/compromised/domains/live/
https://github.com/jarelllama/Scam-Blocklist/blob/main/lists/wildcard_domains/scams.txt
https://raw.githubusercontent.com/mitchellkrogza/Phishing.Database/master/phishing-domains-ACTIVE.txt
https://azorult-tracker.net/api/list/domain?format=plain
https://hosts.tweedge.net/malicious.txt (viene reinserito in quanto alcuni domini contenenti malware non risultano presenti sui vari aggregatori)
http://phishing.mailscanner.info/phishing.bad.sites.conf (Torna on-line e funzionante in caso di malfunzionamento verrà eliminato nuovamente)
09/01/2025 AGGIUNGERE https://curbengh.github.io/malware-filter/urlhaus-filter-online.txt
09/01/2025 AGGIUNGERE https://gitlab.com/curben/urlhaus-filter/-/raw/master/urlhaus-filter-domains.txt
09/01/2025 AGGIUNGERE https://iplists.firehol.org/files/firehol_level1.netset
09/01/2025 AGGIUNGERE https://malware-filter.gitlab.io/malware-filter/phishing-filter-hosts.txt
09/01/2025 AGGIUNGERE https://myip.ms/files/blacklist/general/latest_blacklist.txt
09/01/2025 AGGIUNGERE https://raw.githubusercontent.com/deathbybandaid/piholeparser/master/Subscribable-Lists/ParsedBlacklists/ImmortalMalwareDomains.txt
09/01/2025 AGGIUNGERE https://raw.githubusercontent.com/hagezi/dns-blocklists/main/domains/tif.txt
09/01/2025 AGGIUNGERE https://raw.githubusercontent.com/iam-py-test/my_filters_001/main/Alternative%20list%20formats/antimalware_hosts.txt


2) Liste per il blocco delle pubblicità

Queste liste effettuano il solo blocco delle pubblicità presenti sui siti web, nei programmi installati su pc e funzionano anche per le app installate su Smartphone.

Nota: consiglio di associare le liste dell’elenco che segue al gruppo con il nome Pubblicità.

https://raw.githubusercontent.com/anudeepND/blacklist/master/adservers.txt
https://s3.amazonaws.com/lists.disconnect.me/simple_ad.txt
https://raw.githubusercontent.com/FadeMind/hosts.extras/master/UncheckyAds/hosts
https://v.firebog.net/hosts/Admiral.txt
https://pgl.yoyo.org/adservers/serverlist.php?hostformat=hosts&showintro=0&mimetype=plaintext
https://raw.githubusercontent.com/bigdargon/hostsVN/master/hosts
https://raw.githubusercontent.com/r-a-y/mobile-hosts/master/AdguardMobileAds.txt
https://raw.githubusercontent.com/d43m0nhLInt3r/socialblocklists/master/MobileAppAds/appadsblocklist.txt
https://winhelp2002.mvps.org/hosts.txt
https://v.firebog.net/hosts/static/w3kbl.txt
https://raw.githubusercontent.com/kboghdady/youTube_ads_4_pi-hole/master/youtubelist.txt
https://raw.githubusercontent.com/CitizenXVIL/Hosts/master/Youtube%20hosts.txt
https://blocklistproject.github.io/Lists/youtube.txt
https://blocklistproject.github.io/Lists/ads.txt
https://someonewhocares.org/hosts/zero/hosts
https://www.sunshine.it/blacklist.txt
https://adaway.org/hosts.txt
https://v.firebog.net/hosts/AdguardDNS.txt
https://raw.githubusercontent.com/soteria-nou/domain-list/master/ads.txt
https://www.technoy.de/lists/blocklist.txt
09/01/2025 AGGIUNGERE https://cdn.jsdelivr.net/gh/badmojr/1Hosts@latest/Lite/domains.txt
09/01/2025 AGGIUNGERE (Nota: da aggiungere se non usate il filtro famiglia): https://hosts.ubuntu101.co.za/domains.list


3) Liste da aggiungere per il blocco di Tracking/Telemetria

Queste liste evitano il tracciamento utente, la profilazione e la telemetria anche per programmi e app installate su Cellulare/Smartphone.

Nota: consiglio di associare le liste dell’elenco che segue al gruppo con il nome Tracking_Telemetria.

https://raw.githubusercontent.com/FadeMind/hosts.extras/master/add.2o7Net/hosts
https://v.firebog.net/hosts/Easyprivacy.txt
https://v.firebog.net/hosts/Prigent-Ads.txt
https://gitlab.com/quidsup/notrack-blocklists/raw/master/notrack-blocklist.txt
https://hostfiles.frogeye.fr/firstparty-trackers-hosts.txt
https://raw.githubusercontent.com/hagezi/dns-blocklists/main/adblock/light.txt
https://www.github.developerdan.com/hosts/lists/amp-hosts-extended.txt
https://blocklistproject.github.io/Lists/tracking.txt
https://www.github.developerdan.com/hosts/lists/tracking-aggressive-extended.txt
!Attenzione!: Questa è una lista aggressiva e serve a bloccare il tracciamento e targeting geografico, potrebbe bloccare qualche funzionalità di qualche sito web o servizio, si prega quindi di non usarla, se non si è disposti a esaminare i domini bloccati nella scheda query log e inserirli nella propria whitelist.
4) Liste per tracking/telemetria server Microsoft (Windows/Office/etc.)

Ho deciso di creare questa “nuova” sezione, dedicata al mondo “Microsoft” in modo che sia ben visibile ai lettori e possano scegliere liberamente se avvalersene o meno. Si segnala che esistono valide alternative a Microsoft, per esempio invece di usare il pacchetto Microsoft Office a pagamento, è possibile usufruire di prodotti completamente free ed open source, come ad esempio: LibreOffice, OpenOffice, etc.

Si segnala che tutte queste liste sono valide sia per Pc (Windows, Linux, Mac) che per cellulare/smartphone (Android e Apple Iphone) e possono effettuare il blocco di:

Login sul web che sfruttano i servizi Microsoft (email, azure, Microsoft office sul web, etc.).
Applicazioni del sistema operativo Windows.
Pacchetto Microsoft Office qualsiasi versione (su PC, Mac, Cellulare Android e Apple Iphone).
Servizi e Aggiornamenti dei prodotti di Microsoft (sia del sistema operativo che del pacchetto Office quindi su PC, Android, Apple Iphone e Mac).
PARTE 1 – Liste per blocco tracking e telemetria Microsoft:

Nota: consiglio di associare le liste dell’elenco che segue al gruppo con il nome: Microsoft.

https://raw.githubusercontent.com/crazy-max/WindowsSpyBlocker/master/data/hosts/spy.txt
https://raw.githubusercontent.com/crazy-max/WindowsSpyBlocker/master/data/hosts/spy_v6.txt (versione ipv6)
https://raw.githubusercontent.com/RPiList/specials/master/Blocklisten/Win10Telemetry
https://raw.githubusercontent.com/RPiList/specials/master/Blocklisten/MS-Office-Telemetry
https://raw.githubusercontent.com/hagezi/dns-blocklists/main/adblock/native.winoffice.txt
09/01/2025 AGGIUNGERE https://raw.githubusercontent.com/Loyalsoldier/v2ray-rules-dat/release/win-spy.txt
PARTE 2 – Liste per blocco server/siti web di Microsoft e relativi update (aggiornamenti) del sistema operativo Windows, Microsoft Office etc., sono consigliati alla sola utenza esperta:

Nota: consiglio di associare le liste dell’elenco che segue al gruppo con il nome: Microsoft_blocco_totale

Se volete usufruire dei servizi Microsoft più in basso sono presenti vari (non tutti) server/domini da inserire nella whitelist, quindi se siete alle prime armi con il pihole vi sconsiglio di inserire tali liste, bisogna aver appreso un pò d’informazioni ed esperienza, per esaminare il traffico, guardando la scheda Query Log sul pihole e inserire in whitelist i domini e i servizi dove riscontrare anomalie.

– !Attenzione! – Da inserire se si vogliono bloccare gli update di Microsoft (windows/office):

https://raw.githubusercontent.com/crazy-max/WindowsSpyBlocker/master/data/hosts/update.txt
https://raw.githubusercontent.com/crazy-max/WindowsSpyBlocker/master/data/hosts/update_v6.txt (Versione ipv6)
09/01/2025 AGGIUNGERE https://raw.githubusercontent.com/Loyalsoldier/v2ray-rules-dat/release/win-update.txt
– !Attenzione! – Questi filtri bloccano completamente i seguenti servizi e le applicazioni di Microsoft come ad esempio Skype, Bing, Live, Outlook, NCSI, Microsoft Office…, sia su pc, che su cellulare, compreso il sito web outlook.live.com (per leggere le email, esempio indirizzi: xxx@hotmail.it/.com xxx@outlook.it/.com):

https://raw.githubusercontent.com/crazy-max/WindowsSpyBlocker/master/data/hosts/extra.txt
https://raw.githubusercontent.com/crazy-max/WindowsSpyBlocker/refs/heads/master/data/hosts/extra_v6.txt (Versione ipv6)
!Attenzione! – Questa lista effettua il blocco (anche non Microsoft) di: Bing, Outlook, Office, Edge, Skype, Xbox, Microsoft.com, Windows Update, Defender Update, Azure, OneDrive, Spotify, TikTok, Clipchamp, Disney+ , Facebook, Linkedin e infine blocca i server della Telemetria sia del pacchetto Microsoft Office sia che del sistema operativo Windows 10 e Windows 11:

https://raw.githubusercontent.com/schrebra/Windows.10.DNS.Block.List/main/hosts.txt
5) Liste che effettuano un blocco multiplo (pubblicità, tracking, malware, telemetria, evitano anti-adblock su alcuni siti web, etc)

Queste liste effettuano un lavoro a più fattori, quindi un blocco multiplo bloccando: pubblicità, tracking, malware, telemetria, evitano anti-adblock su alcuni siti web, etc.

Nota: consiglio di associare le liste dell’elenco che segue al gruppo con il nome Blocco_multiplo_tracking_e_pubblicità

https://raw.githubusercontent.com/MajkiIT/polish-ads-filter/master/polish-pihole-filters/hostfile.txt
https://raw.githubusercontent.com/neodevpro/neodevhost/master/host
https://big.oisd.nl/
https://raw.githubusercontent.com/StevenBlack/hosts/master/hosts
https://www.github.developerdan.com/hosts/lists/ads-and-tracking-extended.txt
https://v.firebog.net/hosts/Easylist.txt
https://ceadd.ca/blockyoux.txt
https://hblock.molinero.dev/hosts
https://raw.githubusercontent.com/jerryn70/GoodbyeAds/master/Hosts/GoodbyeAds.txt
https://raw.githubusercontent.com/RooneyMcNibNug/pihole-stuff/master/SNAFU.txt
https://sebsauvage.net/hosts/hosts
https://raw.githubusercontent.com/notracking/hosts-blocklists/master/hostnames.txt
https://raw.githubusercontent.com/lassekongo83/Frellwits-filter-lists/master/Frellwits-Swedish-Hosts-File.txt
https://raw.githubusercontent.com/bongochong/CombinedPrivacyBlockLists/master/newhosts-final.hosts
https://raw.githubusercontent.com/infinitytec/blocklists/master/ads-and-trackers.txt
https://www.technoy.de/lists/Suspicious.txt
https://www.technoy.de/lists/fake-streaming.txt
6) Liste da usare/aggiungere se si possiede: una Smart Tv, oppure una Firestick, oppure un box Android per la TV

Tali liste effettuano un blocco sulla pubblicità e bloccano il tracciamento, telemetria e profilazione effettuata dai produttori delle smart tv

Nota: consiglio di associare le liste dell’elenco che segue al gruppo con il nome Smart_Tv.

https://blocklistproject.github.io/Lists/smart-tv.txt
https://raw.githubusercontent.com/d43m0nhLInt3r/socialblocklists/master/SmartTV/smarttvblocklist.txt
https://raw.githubusercontent.com/Perflyst/PiHoleBlocklist/master/SmartTV.txt
https://blocklistproject.github.io/Lists/basic.txt
https://raw.githubusercontent.com/RPiList/specials/master/Blocklisten/samsung (Nota: attenzione può creare blocchi sul acuni siti web)
https://gist.githubusercontent.com/eterps/9ddb13a118a21a7d9c12c6165e0bbff5/raw/0ba4b04802a4b478d7777fb7abe76c8eac0c5bfc/Samsung%2520Smart-TV%2520Blocklist%2520Adlist%2520(for%2520PiHole)
https://raw.githubusercontent.com/Perflyst/PiHoleBlocklist/master/AmazonFireTV.txt
7) Liste per bloccare programmi e script mining (che possono attaccare il browser: Firefox, Chrome, Opera, etc)

Tali liste effettuano un blocco sui programmi e script mining (di base sono script in linguaggio java che possono attaccare anche il browser) fanno guadagnare soldi agli hacker a scapito di risorse utilizzare in modo inappropriato sui nostri device (ad esempio: può capitare che il nostro pc abbia un uso anomalo e spropositato di processore e ram, causando un calo di prestazioni anche elevato, oltre che un consumo di elettricità più elevato).

Nota: consiglio di associare le liste dell’elenco che segue al gruppo con il nome: Webminer_e_Coin

UPDATE 07/01/2025 RIMUOVERE https://zerodot1.gitlab.io/CoinBlockerLists/hosts_browser Al momento risulta essere fuori servizio (si ringrazia Francesco per la segnalazione)
UPDATE 07/01/2025 AGGIUNGERE https://raw.githubusercontent.com/Ultimate-Hosts-Blacklist/ZeroDot1_CoinBlockerLists/refs/heads/master/domains.list (in sostituzione di zerodot1.gitlab.io/CoinBlockerLists/hosts_browser)
https://raw.githubusercontent.com/hoshsadiq/adblock-nocoin-list/master/hosts.txt
https://raw.githubusercontent.com/anudeepND/blacklist/master/CoinMiner.txt
https://raw.githubusercontent.com/austinheap/sophos-xg-block-lists/master/nocoin.txt
UPDATE 07/01/2025 https://github.com/stamparm/maltrail/blob/master/trails/static/suspicious/crypto_mining.txt
UPDATE 07/01/2025 https://raw.githubusercontent.com/hoshsadiq/adblock-nocoin-list/refs/heads/master/nocoin.txt

8) Liste Parental Control – Filtro Famiglia

Le liste Parental Control o Filro Famiglia, aiutano i genitori a nascondere le minaccie e i pericoli presenti sul web, proteggendo i bambini che risultano “ormai” sempre connessi alla rete. Facendone uso si evitano i siti web per: adulti(porno), giochi d’azzardo, droga, tutti i social (facebook, tiktok, twitter, instagram, skype, etc.) e anche i download illegali.

Nota: consiglio di associare le liste dell’elenco che segue al gruppo con il nome Parental_Control, Questo permette anche di aggregare il gruppo a uno o più determinati dispositivi (specifico tablet/smartphone/etc).

The Ultimate Hosts Blacklist – Filtro di blocco multiplo (pubblicità, telemetria, social, porno, giochi d’azzardo, etc.):

https://hosts.ubuntu101.co.za/domains.list (Nota: tale filtro può essere inserito anche nel blocco pubblicità se non usate il filtro famiglia)

Blocco siti per adulti:

https://raw.githubusercontent.com/mhhakim/pihole-blocklist/master/porn.txt
https://raw.githubusercontent.com/chadmayfield/pihole-blocklists/master/lists/pi_blocklist_porn_top1m.list
https://raw.githubusercontent.com/chadmayfield/my-pihole-blocklists/master/lists/pi_blocklist_porn_all.list
https://blocklistproject.github.io/Lists/porn.txt
https://blocklistproject.github.io/Lists/abuse.txt
https://raw.githubusercontent.com/RPiList/specials/master/Blocklisten/pornblock1
https://raw.githubusercontent.com/RPiList/specials/master/Blocklisten/pornblock2
https://raw.githubusercontent.com/RPiList/specials/master/Blocklisten/pornblock3
https://raw.githubusercontent.com/RPiList/specials/master/Blocklisten/pornblock4
https://v.firebog.net/hosts/Prigent-Adult.txt
https://www.technoy.de/lists/xporn.txt
https://nsfw.oisd.nl
Blocco siti che trattano gioco d’azzardo, droga e torrent:

https://blocklistproject.github.io/Lists/gambling.txt
https://blocklistproject.github.io/Lists/drugs.txt
https://blocklistproject.github.io/Lists/torrent.txt
Blocco social (esempio: facebook, tiktok, twitter, instagram, skype, reddit etc.):

https://blocklistproject.github.io/Lists/tiktok.txt
https://raw.githubusercontent.com/infinitytec/blocklists/master/tiktok.txt
https://blocklistproject.github.io/Lists/facebook.txt
https://raw.githubusercontent.com/anudeepND/blacklist/master/facebook.txt
https://github.com/d43m0nhLInt3r/socialblocklists/blob/master/Snapchat/snapchatblocklist.txt
https://github.com/d43m0nhLInt3r/socialblocklists/blob/master/Skype/skypeblocklist.txt
https://raw.githubusercontent.com/infinitytec/blocklists/master/reddit.txt
9) Liste per utenti esperti

Queste liste effettuano un blocco completo sul tracciamento utente, traffico analitico oltre a pubblicità e malware, e possono causare problemi di navigazione. Per poterle inserire, dovete saper gestire il pihole, esaminare il traffico, guardando la scheda Query Log e inserire in whitelist i domini che volete visitare e che risultano bloccati dai filtri, in modo da poter navigare sui siti web dove riscontrate problemi/anomalie.

Nota: consiglio di associare le liste dell’elenco che segue al gruppo con il nome: Filtri_per_utenti_esperti_opzionali.

https://raw.githubusercontent.com/parseword/nolovia/master/skel/hosts-nolovia.txt
!Attenzione!: Questo filtro blocca molte connessioni verso alcuni server che vengono usati per effettuare il login tramite Google/Facebook etc, quindi alcuni siti non funzionano, per risolvere è necessario intervenire sulle whitelist, se non lo sapete fare, non inseritelo.
https://raw.githubusercontent.com/hagezi/dns-blocklists/main/adblock/multi.txt
!Attenzione!: blocca tutti i siti e domini che effettuano tracciamento, anche di tipo analitico come ad esempio il dominio di adobe, questo provoca errori sul funzionamento delle applicazioni installate come Adobe Acrobat Reader/Writer, per risolvere è necessario intervenire sulle whitelist.
UPDATE 09/01/2025 https://raw.githubusercontent.com/Loyalsoldier/v2ray-rules-dat/release/reject-list.txt


-->