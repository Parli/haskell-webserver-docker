FROM haskell:8.6.5 as buildenv

WORKDIR /app

RUN stack upgrade --binary-version 2.1.1
RUN stack update
RUN stack setup --resolver lts-13.26

RUN stack build Cabal
RUN stack build haskell-src-exts
RUN stack build lens
RUN stack build aeson
RUN stack build http-conduit
RUN stack build servant-server
