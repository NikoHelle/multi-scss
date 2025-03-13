# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [1.0.0] - Mar, 14, 2025

### Added

#### Selector middleware

Added `config.selectorMiddware` which is called everytime a selector is created.

### Removed

#### Automatic alias system

The `alias` argument is not stored anymore and automatic checks and conversions for selector aliases have been removed. There is a migration function `alias-middleware` which does the same thing.

#### Alias related functions

All `*-alias-*` functions removed.
