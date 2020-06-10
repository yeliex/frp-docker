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
    FRPC_ADMIN_USER=admin \
    FRPC_ADMIN_PWD=password

EXPOSE 7400

RUN mkdir -p /app/config && \
    mkdir -p /app/log && \
    ln -sf /dev/stdout /app/log/frpc.log

WORKDIR /app

COPY --from=build /build/frpc ./

COPY --from=build /build/frpc_full.ini ./config/

COPY ./frpc.ini ./config/

CMD ./frpc -c ./config/frpc.ini
