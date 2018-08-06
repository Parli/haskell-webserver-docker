FROM haskell:8.2.1 as buildenv
WORKDIR /app
RUN apt-get update && apt-get -y install make xz-utils libpq-dev
RUN stack upgrade --binary-version 1.6.1
RUN stack update && stack setup --resolver lts-11.18
RUN stack build Cabal --resolver lts-11.18
RUN stack build haskell-src-exts --resolver lts-11.18
RUN stack build lens --resolver lts-11.18
RUN stack build aeson --resolver lts-11.18
RUN stack build http-conduit --resolver lts-11.18
RUN stack build servant-server --resolver lts-11.18
