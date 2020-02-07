# AskSin Analyzer XS

Funktelegramm-Dekodierer für den Einsatz in HomeMatic Umgebungen als Desktop-App.

Der AskSin Analyzer XS ist eine alternative Implementierung des [AskSinAnalyzer](https://github.com/jp112sdl/AskSinAnalyzer) ohne ESP32 und Display was die Umsetzung vereinfacht.

![AskSinAnalyzerXS-TelegramList](./docs/AskSinAnalyzerXS-TelegramList.png)

## AskSinSniffer328P Hardware

* Arduino Pro Mini 8Mhz 3.3V
* CC1101 Funkmodul
* USB-UART Adapter (FTDI, CP2102, etc)

Der Aufbau folgt der [allgemeingültige Verdrahtung des Pro Mini mit dem CC1101 Funkmodul](https://asksinpp.de/Grundlagen/01_hardware.html#verdrahtung). Der Config-Taster findet keine Verwendung und die Status-LED ist optional. 

Die Daten des AskSinSniffer328P werden über einen USB-UART Adapter an den AskSinAnalyzerXS übertragen und dort ausgewertet und visualisiert.

![AskSinAnalyzerXS-Settings](./docs/AskSinAnalyzerXS-Settings.png)


## Installation

* [Latest release](https://github.com/psi-4ward/AskSinAnalyzerXS/releases/latest)
* [Develop release](https://github.com/psi-4ward/AskSinAnalyzerXS/releases/tag/0.0.0) 0.0.0

### AVR Sketch

Auf dem ATmega328P wird der [AskSinSniffer328P-Sketch](https://github.com/jp112sdl/AskSinAnalyzer/tree/master/AskSinSniffer328P) geflasht. Das Vorgehen ist auf [asksinpp.de](https://asksinpp.de/Grundlagen/) erläutert.

### Electron-App

Die Desktop-Anwendung steht für Windows, MacOS und Linux zum Download unter [Releases](https://github.com/psi-4ward/AskSinAnalyzerXS/releases) bereit.

Tipp: Der AskSinAnalyzerXS gibt einige Debug-Informationen auf der Commando-Zeile aus. Bei Problemen empfiehlt sich also ein Start über ein Terminal. (Bash, cmd).

### Node-App

Der AskSinAnalyzerXS kann auch als Node.js Anwendung betrieben werden was z.B. auf einem Server sinnvoll sein kann.

```bash
$ npm i -g asksin-analyzer-xs
$ asksin-analyzer-xs
Detected SerialPort: /dev/ttyUSB0 (FTDI)
Server started on port 8081
```

Die WebUI kann über den Browser auf [http://localhost:8081](http://localhost:8081) aufgerufen werden.

* Der develop-build des master-Branch ist **nicht** als npm-Paket verfügbar.

## Konfiguration

### Auflösung von Gerätenamen

Der AskSinSniffer328P sieht nur die _Device-Addresses_, nicht aber deren Seriennummern oder Namen. Damit die Adressen in Klartextnamen aufgelöst werden können muss eine DeviceListe von der CCU geladen werden wofür ein Script auf der CCU nötig ist. Siehe [AskSinAnalyzer CCU Untersützung](https://github.com/jp112sdl/AskSinAnalyzer/wiki/CCU_Unterst%C3%BCtzung).

Soll die Geräteliste von FHEM abgerufen werden ist in der `99_myUtils.pm` folgende Funktion einzufügen:

```ruby
sub printHMDevs {
  my @data;
  foreach my $device (devspec2array("TYPE=CUL_HM")) {
    my $snr = AttrVal($device,'serialNr','');
	$snr = "<Zentrale>" if AttrVal($device,'model','') eq 'CCU-FHEM';
	if( $snr ne '' ) {
	  my $name = AttrVal($device,'alias',$device);
	  my $addr = InternalVal($device,'DEF','0');
	  push @data, { name => $name, serial => $snr, address => hex($addr) };
	}
  }
  return JSON->new->encode( { created => time, devices => \@data } );
}
```

Im AskSinAnalyzerXS ist bei Verwendung vom FHEM die Option `Device-List Backend ist eine CCU` zu deaktivieren und als `Device-List URL` wird der Wert `http://fhem.local:8083/fhem?cmd={printHMDevs()}&XHR=1` eingetragen.

## Lizenz

CC BY-NC-SA 4.0
