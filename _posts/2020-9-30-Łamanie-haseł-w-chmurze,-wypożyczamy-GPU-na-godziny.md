---
layout: post
title: Łamanie haseł w chmurze, wypożyczamy GPU na godziny!
---

Na pewno wielu początkujących pentesterów nie ma pod ręką mocy obliczeniowej, pozwalającą na swobodne łamanie haseł. Jak tanio stworzyć takie środowisko gdy zaczynamy przygodę z hackingiem/pentestami i nie mamy paru tysięcy na karty graficzne?

### Wprowadzenie
Z pomocą przychodzi nam serwis [vast.ai](https://vast.ai) - "wypożyczalnia" GPU na godziny, oparta na bazie dockerów. Możemy korzystać z usługi jako host, zarabiając na wynajmie naszego sprzętu (nie korzystałem) lub jako klient wynająć dostępne z listy maszyny. W porównaniu do konkurencji oferującej moc obliczeniową w chmurze, cena jest bardzo niska - przykładowo 1x RTX 2080 Ti na godzinę kosztuje ~20 centów. 
Największe plusy tego rozwiązania to: 
* cena 
* jeden prosty interfejs
* szybki czas konfiguracji środowiska.

Od założenia konta do pierwszego crackowania dzieli nas maksymalnie 5 - 10 minut. Wszystko było testowane pod systemem Kali Linux, ale to samo można bez problemu zrobić na Windowsie. Potrzebujemy tylko jakiegoś klienta ssh, np. PuTTY.

![ceny]({{ site.baseurl }}/images/vast.ai-prices.png)


### Konfiguracja
1. Zakładamy konto na vast.ai i dokonujemy wpłaty. Podczas gdy ja się rejestrowałem był nawet jakiś mały gratis za założenie konta (chyba 1 dolar), więc może i Wam się poszczęści. 
2. Na sam początek generujemy na swoim systemie parę kluczy SSH poleceniem:
`ssh-keygen -t rsa`
3. Kopiujemy nasz klucz publiczny: 
`cat ~/.ssh/id_rsa.pub`
Wchodzimy na vast.ai -> Account -> Change SSH Key i wklejamy zawartość.
4. Następnie Client -> Create -> Instance Configuration -> EDIT IMAGE & CONFIGURATION. Skorzystamy z niestandardowego dockera z gotowym do odpalenia hashcatem. Zjeżdzamy więc w dół listy i w odpowiednie miejsce wpisujemy 'dizcza/docker-hashcat', natomiast poniżej w "Launch mode" wybieramy SSH. Możemy również skorzystać z dowolnego dockera z tej [listy](https://hub.docker.com/search?q=hashcat&type=image).
![docker]({{ site.baseurl }}/images/docker-hashcat.png)


5. Teraz możemy już wybrać interesujący nas zestaw i zacząć zabawę. Po wypożyczeniu przechodzimy do zakładki "Instances", czekamy około minuty aż się wszystko załaduje i klikamy "connect". Kopiujemy polecenie, u mnie wygląda tak: `ssh -p 19694 root@ssh4.vast.ai -L 8080:localhost:8080` i wklejamy do naszego boxa. Po zatwierdzeniu rozpoczyna się sesja ssh i możemy zaczynać działać.

### Szybki test na hashu md5
Na początek ściągniemy sobie jakiś lekki słownik:

`wget https://raw.githubusercontent.com/danielmiessler/SecLists/master/Passwords/Common-Credentials/10-million-password-list-top-1000.txt`

Następnie wybieramy sobie jakieś proste hasło i generujemy hash md5:

`echo -n password | md5sum`

`5f4dcc3b5aa765d61d8327deb882cf99  -`

`echo 5f4dcc3b5aa765d61d8327deb882cf99 >> hash.txt`

Słowo "password" zapisuje do pliku hash.txt. Teraz pozostało nam już tylko uruchomienie hashcata z opcją łamania MD5.

`hashcat hash.txt -m 0 -o result.txt 10-million-password-list-top-1000.txt `

Złamane hasło możemy podejrzeć w pliku result.txt.
