# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [2.2.0] - 2023-03-30

### Changed

* `gafferJtaJtaTransactionManager` bean name to `transactionManager`.
  We would like the `tw-gaffer-jta-starter` to be fully autoconfiguring the service. Unfortunately, some Spring Boot starter libraries are
  specifically expecting a bean named `transactionManager`.

### Migration Guide

#### Transaction Manager Bean Name

If you were using hardcoded `gafferJtaJtaTransactionManager` bean name in your service, you have to change it now to `transactionManager`.

## [2.1.0] - 2023-02-14

### Added

* Spring Boot auto configuration module `tw-gaffer-jta-starter`.

### Changed

* Connection and DataSource wrappers are now implementing `tw-base-utils`'s wrapper interfaces.
* `com.transferwise.common.gaffer.jdbc.DataSourceImpl` class was renamed to `com.transferwise.common.gaffer.jdbc.GafferJtaDataSource`.

### Migration Guide

#### Auto Configuration for Gradle

It is recommended to use full autoconfiguration, which will set up the transaction manager and instrument all the data sources.

For that

1. Make sure, that all your data sources are exposed as beans.
   If this is not possible, you still need to wrap those data sources into `GafferJtaDataSource`, manually.
2. Remove `tw-gaffer-jta` from `implementation` configuration.
   Unless, you would need manual wrapping of `GafferJtaDataSource`.
3. Remove custom wrappings of `GafferJtaDataSource` / `DataSourceImpl`.
   In Wise context, your data source beans should be just plain `HikariDataSource` instances.
4. Remove the code creating all the beans now defined in the `GafferJtaConfiguration` class.
   In a typical Wise service, it comes down to deleting the whole `TransactionManagerConfiguration` class.
5. Add `tw-gaffer-jta` into `runtimeOnly` configuration.

#### Without Auto Configuration

* Change all `com.transferwise.common.gaffer.jdbc.DataSourceImpl` occurrences to `com.transferwise.common.gaffer.jdbc.GafferJtaDataSource`

## [2.0.0] - 2022-12-13

### Fixed

* Transaction timeout 0 is widely considered as no timeout is applied.
  So we do the same.

  The issue was discovered in one of our service, where one method had `@Transactional(timeout=1)`.
  When Spring exits that method, it sets transaction timeout through `JtaTransactionManager` to `0`, indicating that there
  should not be any timeout for next transactions.

  Gaffer however interpreted this as there is a timeout with duration of 0, and ofc. started to give timeout exceptions for all following
  transactions.

  As a side note, it seems like Spring `JtaTransactionManager` is misbehaving. It would be more logical it to either
  a) restore the timeout value to what it was before entering the method
  b) set it to `-1`, which signal to JTA transaction manager to use default timeout.

### Removed

* JMS Support
  Everyone is using Kafka and also Transactional Outbox pattern. There is not much need to have 1PC
  JTA transactions around database changes and JMS.

### Changed

* Upgraded dependencies to the level of Spring Boot 2.6.

* Created matrix tests for Spring Boot 2.5, 2.6 and 2.7 dependencies.

## [1.5.0] - 2022-05-19

### Changed

* Prevent connection from leaking when any tx associated resource error occurs.

## [1.4.0] - 2021-05-27

### Changed

* Moved from JDK 8 to JDK 11.
* Starting to push to Maven Central again.
