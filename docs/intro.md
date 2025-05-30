---
sidebar_position: 999999
slug: /plu-ts
---

# plu-ts

Built with ❤️ by [Harmonic Laboratories](https://www.harmoniclabs.tech/)

This documentation is for `plu-ts` v0.8.0^, if you are using a previous version it is suggested to install the latest version, feel free to ask for help in the [Harmonic Labs' discord server](https://discord.gg/CGKNcG7ade)

## Introduction

`plu-ts` is a library designed for building Cardano dApps in an efficient and developer friendly way.

It is composed of two main parts:

- `plu-ts/onchain`: an [eDSL](https://en.wikipedia.org/wiki/Domain-specific_language#External_and_Embedded_Domain_Specific_Languages) (embedded Domain Specific Language) that leverages Typescript as the host language; designed to generate efficient Smart Contracts.
- `plu-ts/offchain`: a set of classes and functions that allow reuse of onchain types.

# Getting started

see [Getting Started](./Getting%20Started) to begin your journey in plu-ts

## Design principles

`plu-ts` was designed with the following goals in mind, in order of importance:

- Smart Contract efficiency
- developer experience
- reduced script size
- readability