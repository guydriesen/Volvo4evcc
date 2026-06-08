FROM debian:trixie-slim AS download

ARG TARGETPLATFORM

WORKDIR /root
SHELL ["/bin/bash", "-c"]
RUN  apt-get update \
  && apt-get install -y wget jq \
  && rm -rf /var/lib/apt/lists/* \
  && declare -A archs=([linux/amd64]=x64 [linux/arm64]=arm64 [linux/arm/v7]=arm32) \
  && arch=${archs[${TARGETPLATFORM}]} \
  && release=$(wget -qO - https://api.github.com/repos/PowerShell/PowerShell/releases/latest) \
  && package=$(echo -E ${release} | jq -r ".assets[].browser_download_url" | grep "linux-${arch}.tar.gz") \
  && wget ${package} \
  && mkdir -p /opt/microsoft/powershell/7 \
  && tar -xvf "./${package##*/}" -C /opt/microsoft/powershell/7 \
  && rm "./${package##*/}" \
  && echo

FROM debian:trixie-slim

RUN groupadd --system volvo4evcc && \
    useradd --system --create-home -g volvo4evcc volvo4evcc

RUN  apt-get update \
  && apt-get install --no-install-recommends -y ca-certificates libicu76 \
  && rm -rf /var/lib/apt/lists/* \
  && echo

COPY --from=download /opt/microsoft/powershell/7 /opt/microsoft/powershell/7
RUN ln -s /opt/microsoft/powershell/7/pwsh /usr/bin/pwsh

WORKDIR /opt/Volvo4evcc
COPY Volvo4evcc .

USER volvo4evcc

RUN  mkdir -p /home/volvo4evcc/.config/powershell \
  && echo Import-Module "/opt/Volvo4evcc/Volvo4evcc.psd1" > /home/volvo4evcc/.config/powershell/Microsoft.PowerShell_profile.ps1

WORKDIR /data
VOLUME /data

EXPOSE 6060
ENTRYPOINT ["pwsh", "-c"]
CMD ["Start-Volvo4Evcc"]
