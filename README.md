# Crypto Tracker

Ionic Vue app that lists live cryptocurrency prices from Coinlore with manual and pull-to-refresh updates.

## Features

- Live prices from Coinlore tickers with loading, error, and empty states.
- Toolbar refresh button plus pull-to-refresh gesture.
- Ranks, names, symbols, and coin icons (CoinCap icons with letter fallback).
- Ionic 8 UI with a dark, gradient background.

## Prerequisites

- Node.js 18+ and npm

## Setup

```bash
npm i -g @ionic/cli
npm install
```

## Run the app

```bash
ionic serve
```

## API Notes

- Prices are fetched from `https://api.coinlore.net/api/tickers/` (no API key required).
