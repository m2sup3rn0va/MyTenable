# Tenable.SC One-Liners

> **Owner** : Mr. Sup3rN0va |  02-06-2022

## Pre-requisites

- `sudo apt-get install -y curl jq prips`
- Add this to `.bashrc`, if you may:
  - `export SECRET_KEY="VALUE"`
  - `export ACCESS_KEY="VALUE"`
  - `export SCANNER_IP="IP"`

## One-Liners

### Get Scan Names
```sh
echo "SCAN_NAME" | xargs -I{} bash -c 'curl -s -k -H "x-apikey: accesskey=$ACCESS_KEY; secretkey=$SECRET_KEY" https://$SCANNER_IP/rest/scanResult | jq -r ".response.usable[].name" | grep -i "{}"'
```

### Get Repository Name
```sh
echo "REPO_NAME" | xargs -I{} bash -c 'curl -s -k -H "x-apikey: accesskey=$ACCESS_KEY; secretkey=$SECRET_KEY" https://$SCANNER_IP/rest/repository | jq -r ".response[].name" | grep -i "{}"'
```

### Scanned IPs List
```sh
echo "SCAN_NAME" | xargs -I{} bash -c 'curl -s -k -H "x-apikey: accesskey=$ACCESS_KEY; secretkey=$SECRET_KEY" https://$SCANNER_IP/rest/scanResult | jq ".response.usable[] | select(.name==\"{}\")"' | jq -r '.id' | tail -n 1 | xargs -I{} bash -c 'curl -s -k -H "x-apikey: accesskey=$ACCESS_KEY; secretkey=$SECRET_KEY" https://$SCANNER_IP/rest/scanResult\?id\={}' | jq -r '.response.progress.scannedIPs' | tr "," "\n" | xargs -I{} bash -c 'if echo {} | grep "-"; then tmp={}; prips $(echo "${tmp/-/ }"); else echo {};fi' | grep -v "-" | sort -Vu
```

### Scanned IPs Count From Tenable
```sh
echo "SCAN_NAME" | xargs -I{} bash -c 'curl -s -k -H "x-apikey: accesskey=$ACCESS_KEY; secretkey=$SECRET_KEY" https://$SCANNER_IP/rest/scanResult | jq ".response.usable[] | select(.name==\"{}\")"' | jq -r '.id' | tail -n 1 | xargs -I{} bash -c 'curl -s -k -H "x-apikey: accesskey=$ACCESS_KEY; secretkey=$SECRET_KEY" https://$SCANNER_IP/rest/scanResult\?id\={}' | jq -r '.response.progress.scannedSize'
```

### Scanned IPs Count From script
```sh
echo "SCAN_NAME" | xargs -I{} bash -c 'curl -s -k -H "x-apikey: accesskey=$ACCESS_KEY; secretkey=$SECRET_KEY" https://$SCANNER_IP/rest/scanResult | jq ".response.usable[] | select(.name==\"{}\")"' | jq -r '.id' | tail -n 1 | xargs -I{} bash -c 'curl -s -k -H "x-apikey: accesskey=$ACCESS_KEY; secretkey=$SECRET_KEY" https://$SCANNER_IP/rest/scanResult\?id\={}' | jq -r '.response.progress.scannedIPs' | tr "," "\n" | xargs -I{} bash -c 'if echo {} | grep "-"; then tmp={}; prips $(echo "${tmp/-/ }"); else echo {};fi' | grep -v "-" | sort -Vu | wc -l
```

### Dead IPs List
```sh
echo "SCAN_NAME" | xargs -I{} bash -c 'curl -s -k -H "x-apikey: accesskey=$ACCESS_KEY; secretkey=$SECRET_KEY" https://$SCANNER_IP/rest/scanResult | jq ".response.usable[] | select(.name==\"{}\")"' | jq -r '.id' | tail -n 1 | xargs -I{} bash -c 'curl -s -k -H "x-apikey: accesskey=$ACCESS_KEY; secretkey=$SECRET_KEY" https://$SCANNER_IP/rest/scanResult\?id\={}' | jq -r '.response.progress.deadHostIPs' | tr "," "\n" | xargs -I{} bash -c 'if echo "{}" | grep "-" > /dev/null; then ip={}; prips $(echo "${ip/-/ }"); elif echo "{}" | grep "|" > /dev/null; then echo "{}" | tr "|" "\n" | grep -v [a-z]; else echo "{}"; fi' | grep -v "-" | sort -Vu
```

### Dead IPs Count From Tenable
```sh
echo "SCAN_NAME" | xargs -I{} bash -c 'curl -s -k -H "x-apikey: accesskey=$ACCESS_KEY; secretkey=$SECRET_KEY" https://$SCANNER_IP/rest/scanResult | jq ".response.usable[] | select(.name==\"{}\")"' | jq -r '.id' | tail -n 1 | xargs -I{} bash -c 'curl -s -k -H "x-apikey: accesskey=$ACCESS_KEY; secretkey=$SECRET_KEY" https://$SCANNER_IP/rest/scanResult\?id\={}' | jq -r '.response.progress.deadHostSize'
```

### Dead IPs Count From Script
```sh
echo "SCAN_NAME" | xargs -I{} bash -c 'curl -s -k -H "x-apikey: accesskey=$ACCESS_KEY; secretkey=$SECRET_KEY" https://$SCANNER_IP/rest/scanResult | jq ".response.usable[] | select(.name==\"{}\")"' | jq -r '.id' | tail -n 1 | xargs -I{} bash -c 'curl -s -k -H "x-apikey: accesskey=$ACCESS_KEY; secretkey=$SECRET_KEY" https://$SCANNER_IP/rest/scanResult\?id\={}' | jq -r '.response.progress.deadHostIPs' | tr "," "\n" | xargs -I{} bash -c 'if echo "{}" | grep "-" > /dev/null; then ip={}; prips $(echo "${ip/-/ }"); elif echo "{}" | grep "|" > /dev/null; then echo "{}" | tr "|" "\n" | grep -v [a-z]; else echo "{}"; fi' | grep -v "-" | sort -Vu | wc -l
```

### ScanIDs for the Scans from a Time Range
```sh
SCAN_NAME="<SCAN_NAME_WITH_SPACES>"; START_DATE="MM/DD/YYYY"; TIME_ZONE="<YOUR_TIMEZONE>";START_TIME=$(date -d "$START_DATE 00:00:00" +%s); tempDate=$(mktemp); tempData=$(mktemp); curl -s -k -X GET -H "x-apikey: accesskey=$ACCESS_KEY; secretkey=$SECRET_KEY" https://$SCANNER_IP/rest/scanResult\?startTime\=$START_TIME\&filter\=usable\&fields\=name,id,importStart | jq ".response.usable[] | select( .name == \"$SCAN_NAME\")" | jq -r ".importStart" | xargs -I{} bash -c 'TZ="$TIME_ZONE" jq -n "{} | strftime(\"%B %d %Y %I:%M%p %Z\")"' > $tempDate; curl -s -k -X GET -H "x-apikey: accesskey=$ACCESS_KEY; secretkey=$SECRET_KEY" https://$SCANNER_IP/rest/scanResult\?startTime\=$START_TIME\&filter\=usable\&fields\=name,id,importStart | jq ".response.usable[] | select( .name == \"$SCAN_NAME\")" | jq -r ". | [.name, .id] | @tsv" > $tempData; paste $tempData $tempDate; rm -rf $tempData $tempDate
```

---
