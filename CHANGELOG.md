## [1.0.3](https://gitlab.com/1O/vaultwarden-backup/compare/v1.0.2...v1.0.3) (2022-02-04)


### Bug Fixes

* deprecation check for $LOGFILE ([11d6cf9](https://gitlab.com/1O/vaultwarden-backup/commit/11d6cf93b0e5b2ea1de84932b84aacd83e40f0c4))
* Moved deprecation checks to set-env script ([81f6190](https://gitlab.com/1O/vaultwarden-backup/commit/81f619001e1028ea6aad372237c22bb43bded045)), closes [#23](https://gitlab.com/1O/vaultwarden-backup/issues/23)
* Set LOG_LEVEL first ([d5ee0b1](https://gitlab.com/1O/vaultwarden-backup/commit/d5ee0b1c60ba5d3c8aab57a6abffbf70f968a72d)), closes [#23](https://gitlab.com/1O/vaultwarden-backup/issues/23)

## [1.0.2](https://gitlab.com/1O/vaultwarden-backup/compare/v1.0.1...v1.0.2) (2022-01-31)


### Bug Fixes

* always create logfiles as $UID:$GID ([0034bfb](https://gitlab.com/1O/vaultwarden-backup/commit/0034bfb70107daf3a70039320ac18a57a85d55f6)), closes [#21](https://gitlab.com/1O/vaultwarden-backup/issues/21)
* Fixed permissions issues with log files ([fec55cc](https://gitlab.com/1O/vaultwarden-backup/commit/fec55cce32790cc23909922f1b1f6ef88dd87b9c)), closes [#21](https://gitlab.com/1O/vaultwarden-backup/issues/21)

## [1.0.1](https://gitlab.com/1O/vaultwarden-backup/compare/v1.0.0...v1.0.1) (2022-01-29)


### Bug Fixes

* Fixed DELETE_AFTER ([b945105](https://gitlab.com/1O/vaultwarden-backup/commit/b945105fd620a07b6a7f513cad9c250e52afab08))
* Fixed permission issues ([6108810](https://gitlab.com/1O/vaultwarden-backup/commit/610881056f6ed89bf6cf30b645121f9edd5ddfb5)), closes [#19](https://gitlab.com/1O/vaultwarden-backup/issues/19) [#20](https://gitlab.com/1O/vaultwarden-backup/issues/20) [#21](https://gitlab.com/1O/vaultwarden-backup/issues/21)
* Fixed su-exec issues in manual mode ([62f9db9](https://gitlab.com/1O/vaultwarden-backup/commit/62f9db996236e649ce07144d056dfae2e2a59dbf)), closes [#20](https://gitlab.com/1O/vaultwarden-backup/issues/20)
* loop when run with UID zero ([724dd65](https://gitlab.com/1O/vaultwarden-backup/commit/724dd657ccb41caec2b0a298c19a28be3caa21dd))

# [1.0.0](https://gitlab.com/1O/vaultwarden-backup/compare/v0.0.8...v1.0.0) (2022-01-26)


### Features

* renamed to vaultwarden-backup ([4ce504e](https://gitlab.com/1O/vaultwarden-backup/commit/4ce504e6debb6cd3993a9f36135b6db539f01dd5)), closes [#18](https://gitlab.com/1O/vaultwarden-backup/issues/18)


### BREAKING CHANGES

* Changed environment variables
