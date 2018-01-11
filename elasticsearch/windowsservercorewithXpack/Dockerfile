# escape=`
FROM openjdk:8-jdk-windowsservercore-ltsc2016
SHELL ["powershell", "-Command", "$ErrorActionPreference = 'Stop'; $ProgressPreference = 'SilentlyContinue';"]

# Elasticsearch
ENV ES_VERSION="6.1.1" `
    ES_SHA1="58c90d988adac0b00ea4f2936aaae96fae63a2f00c2c1617efe53f86eccad669f92f7cec691dde81592ff3228922f98619289af103e125bf9c2ad6e1f7c16700" `
    ES_HOME="C:\elasticsearch"

RUN Invoke-WebRequest -outfile elasticsearch.zip "https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-$($env:ES_VERSION).zip" -UseBasicParsing; `
    #if ((Get-FileHash elasticsearch.zip -Algorithm sha1).Hash -ne $env:ES_SHA1) {exit 1} ; `
    Expand-Archive elasticsearch.zip -DestinationPath C:\ ; `
    Move-Item c:/elasticsearch-$($env:ES_VERSION) $env:ES_HOME; `
    Remove-Item elasticsearch.zip

VOLUME C:\data

# REMARKS - DNS and mount tweaks needed for Windows
RUN Set-ItemProperty -path 'HKLM:\SYSTEM\CurrentControlSet\Services\Dnscache\Parameters' -Name ServerPriorityTimeLimit -Value 0 -Type DWord; `
    Set-ItemProperty -path 'HKLM:\SYSTEM\CurrentControlSet\Control\Session Manager\DOS Devices' -Name 'G:' -Value '\??\C:\data' -Type String    

HEALTHCHECK --interval=10s `
 CMD powershell -command `
    try { `
     $content = (iwr -useb http://localhost:9200/_cat/health).Content; `
     $health = $content.Split(' ')[3]; `
     if ($health -eq 'green' -or $health -eq 'yellow') { return 0 } `
     else { return 1 }; `
    } catch { return 1 }
    
EXPOSE 9200 9300
WORKDIR C:\elasticsearch
COPY config ./config

Run .\bin\elasticsearch-plugin.bat install --batch x-pack

SHELL ["cmd", "/S", "/C"]
CMD ".\bin\elasticsearch.bat"
