FROM alpine:latest as build

RUN mkdir -p /build/tmp

WORKDIR /build

RUN apk update && \
    apk upgrade && \
    ARCH= && alpineArch="$(apk --print-arch)" \
    && case "${alpineArch##*-}" in \
            x86_64) \
              ARCH='amd64' \
              ;; \
            *) ;; \
          esac && \
    apk add jq curl wget tar gzip && \
    git_url="https://api.github.com/repos/fatedier/frp/releases/latest" && \
    release_info=$(curl $git_url) && \
    version=$(echo $release_info | jq ".name" | grep -Eo "(\d+\.?)+") && \
    asset_name="frp_$version""_linux_$ARCH" && \
    asset_bundle_name="$asset_name.tar.gz" && \
    asset_url=$(echo $release_info | jq -c ".assets | map({key: .name, value: .url}) | from_entries | .\"$asset_bundle_name\"") && \
    echo $asset_url && \
    sh -c "wget --header \"accept: application/octet-stream\" -O \"./tmp/frp.tar.gz\" $asset_url" && \
    tar zxvf ./tmp/frp.tar.gz -C ./tmp && \
    cd "./tmp/$asset_name/" && \
    mv * ../../

FROM alpine:latest

ENV FRP_PRIVILEGE_TOKEN=token \
    FRPS_DASHBOARD_USER=admin \
    FRPS_DASHBOARD_PWD=password

EXPOSE 7000 7001 7500

RUN mkdir -p /app/config && \
    mkdir -p /app/log && \
    ln -sf /dev/stdout /app/log/frps.log

WORKDIR /app

COPY --from=build /build/frps ./

COPY --from=build /build/frps_full.ini ./config/

COPY ./frps.ini ./config/

CMD ./frps -c ./config/frps.ini
