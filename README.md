# # RWA Launchpad — Día 3 (Entregable Semana 4, Stellar Elite Bolivia)

> **Video demo:** 

[https://drive.google.com/drive/folders/1Czf0AKNfgCt9loV3U-rPGJ4TZrvJiTZI?usp=sharing](https://drive.google.com/drive/folders/1Czf0AKNfgCt9loV3U-rPGJ4TZrvJiTZI?usp=sharing)

Contrato Soroban del RWA Launchpad con `invest` y `withdraw`, desplegado en **Stellar Testnet**, con herramientas separadas de admin `scripts/admin-tool.sh`) y de usuario `scripts/user-tool.sh`).

## Variación: monto mínimo de inversión

**Cada inversión debe ser de al menos 500 unidades del token de pago.**

| Monto `payment_amount`) | Resultado |

|---|---|

| `< 500` | Falla con `Error(Contract, #7)` → `AmountTooLow` |

| `>= 500` | La inversión continúa normalmente |

## Cambios en el contrato — `src/lib.rs`

**1. Nuevo error** (los códigos 1–6 no cambiaron):

```rust

pub enum Error {

    NotInitialized = 1,

    AlreadyInitialized = 2,

    InsufficientBalance = 3,

    InvalidAmount = 4,

    NotWhitelisted = 5,

    Paused = 6,

    AmountTooLow = 7,

}

```

**2. La validación va en `check_variation_gate`**, que ahora recibe `payment_amount`:

```rust

fn check_variation_gate(env: &Env, investor: &Address, payment_amount: i128) -> Result<(), Error> {

    let _ = (env, investor);

    if payment_amount < 500 {

        return Err(Error::AmountTooLow);

    }

    Ok(())

}

```

**3. `invest` llama al gate** justo después de `require_auth` y antes de verificar la pausa, transferir tokens o mintear. Una inversión inválida se rechaza sin tocar ningún balance.

## Tests — `src/test.rs`

Se agregaron 2 tests usando el setup existente `setup_with_payment_token`) y se mantuvieron los 3 originales.

| Test | Comprueba |

|---|---|

| `test_invest_amount_too_low` | **(nuevo)** `invest(100)` falla específicamente con `AmountTooLow` (vía `try_invest`) |

| `test_invest_minimum_amount` | **(nuevo)** `invest(500)` funciona: devuelve 5 RWA, balance RWA = 5, token de pago 1000 → 500 |

| `test_invest` | Inversión de 500 |

| `test_withdraw` | Retiro de pagos a la tesorería |

| `test_invest_not_whitelisted` | Falla con `#5` si el inversionista no está en whitelist |

```bash

cargo test

# running 5 tests

# test result: ok. 5 passed; 0 failed

```

## Build

```bash

stellar contract build

```

| | |

|---|---|

| WASM | `dia-3/target/wasm32v1-none/release/rwa_launchpad_dia_3.wasm` |

| Tamaño | 7918 bytes |

| Hash | `08db690dfd94f6936d4f403a8827989d4cf0a56f84b87f7c08591e7618b2e6c5` |

| Entorno | Stellar CLI 28.0.0 · soroban-sdk 26 · target `wasm32v1-none` |

## Despliegue en Testnet

### Contratos

| Contrato | ID |

|---|---|

| RWA Launchpad | `CDUEDH4L3KDNIWAPWFRICMHYDEDMGIQ7P65U2GOPFJULOB4PVBUNC66X`]([https://stellar.expert/explorer/testnet/contract/CDUEDH4L3KDNIWAPWFRICMHYDEDMGIQ7P65U2GOPFJULOB4PVBUNC66X](https://stellar.expert/explorer/testnet/contract/CDUEDH4L3KDNIWAPWFRICMHYDEDMGIQ7P65U2GOPFJULOB4PVBUNC66X)) |

| Token de pago (SAC `PAY`) | `CBJ343FZ6RYBSDPF6EDAOAIAK4C7NXOKSWZJALVTL5VJATMD4WEPHYRZ`]([https://stellar.expert/explorer/testnet/contract/CBJ343FZ6RYBSDPF6EDAOAIAK4C7NXOKSWZJALVTL5VJATMD4WEPHYRZ](https://stellar.expert/explorer/testnet/contract/CBJ343FZ6RYBSDPF6EDAOAIAK4C7NXOKSWZJALVTL5VJATMD4WEPHYRZ)) |

El token de pago es un Stellar Asset Contract propio `PAY`, emitido por el admin), usado en lugar del token del instructor. Configuración de `initialize`: `name = RWAToken`, `total_supply = 1000000`, `price_per_unit = 100`.

### Cuentas

| Rol | Identidad CLI | Address |

|---|---|---|

| Admin | `alice` | `GCHKPFDJ44BGFO2O7WQEZT72CHU5L3OKEPVQORT4OI547VHLICSHOFKB`]([https://stellar.expert/explorer/testnet/account/GCHKPFDJ44BGFO2O7WQEZT72CHU5L3OKEPVQORT4OI547VHLICSHOFKB](https://stellar.expert/explorer/testnet/account/GCHKPFDJ44BGFO2O7WQEZT72CHU5L3OKEPVQORT4OI547VHLICSHOFKB)) |

| Investor | `bob` | `GAEKBDGOM45R22OIKGNQCQWMGNDTX5GLGRLB2FH7IG3VUNBHXERE733Z`]([https://stellar.expert/explorer/testnet/account/GAEKBDGOM45R22OIKGNQCQWMGNDTX5GLGRLB2FH7IG3VUNBHXERE733Z](https://stellar.expert/explorer/testnet/account/GAEKBDGOM45R22OIKGNQCQWMGNDTX5GLGRLB2FH7IG3VUNBHXERE733Z)) |

| Tesorería | `treasury` | `GDQUOBHEWF25Z2JYTJBVVLGYSCK3EFJ3Y7DXY3VLHVIIEZNO3LF27DZN`]([https://stellar.expert/explorer/testnet/account/GDQUOBHEWF25Z2JYTJBVVLGYSCK3EFJ3Y7DXY3VLHVIIEZNO3LF27DZN](https://stellar.expert/explorer/testnet/account/GDQUOBHEWF25Z2JYTJBVVLGYSCK3EFJ3Y7DXY3VLHVIIEZNO3LF27DZN)) |

### Setup inicial

| Paso | Transacción |

|---|---|

| Deploy SAC `PAY` | `91b1a3d2…`]([https://stellar.expert/explorer/testnet/tx/91b1a3d27fdbfc13f0086700eec986bbd93d96fd40fdb0910afe4d08a8062edc](https://stellar.expert/explorer/testnet/tx/91b1a3d27fdbfc13f0086700eec986bbd93d96fd40fdb0910afe4d08a8062edc)) |

| Mint 1000 PAY al investor | `9b628ceb…`]([https://stellar.expert/explorer/testnet/tx/9b628cebdc1b96f7d5b06ed20b5fc8d240254b5aa833c8670ad0ae4125ec5002](https://stellar.expert/explorer/testnet/tx/9b628cebdc1b96f7d5b06ed20b5fc8d240254b5aa833c8670ad0ae4125ec5002)) |

| Upload WASM | `bd495374…`]([https://stellar.expert/explorer/testnet/tx/bd49537416cb534024ac795218138e0f925cb712a65f0a79042f9ced7998cd95](https://stellar.expert/explorer/testnet/tx/bd49537416cb534024ac795218138e0f925cb712a65f0a79042f9ced7998cd95)) |

| Deploy launchpad | `1960c8a9…`]([https://stellar.expert/explorer/testnet/tx/1960c8a99a3b386b6661d0f9955a87536b3c54ca4b35e9daf806e6acf4e0dab9](https://stellar.expert/explorer/testnet/tx/1960c8a99a3b386b6661d0f9955a87536b3c54ca4b35e9daf806e6acf4e0dab9)) |

| `initialize` | `515d9461…`]([https://stellar.expert/explorer/testnet/tx/515d9461325546f34ae9136c04582025f4dd51717dde6fae7ba20e58974d71f0](https://stellar.expert/explorer/testnet/tx/515d9461325546f34ae9136c04582025f4dd51717dde6fae7ba20e58974d71f0)) |

| `set_whitelist` | `67f93d5c…`]([https://stellar.expert/explorer/testnet/tx/67f93d5c9861b4914cf875cd37a3ee182262a9512e11f6572cde728241efc9b1](https://stellar.expert/explorer/testnet/tx/67f93d5c9861b4914cf875cd37a3ee182262a9512e11f6572cde728241efc9b1)) |

## Evidencia del entregable

**Inversión de 100 → FALLA con `AmountTooLow`**

```text

❌ error: transaction simulation failed: HostError: Error(Contract, #7)

```

La rechaza la simulación, así que no se envía a la red y no tiene hash.

**Inversión de 500 → ÉXITO**

- Tx hash: `cfd18f9d08a3f213c6751ed372947657df2ef7f5b64a8524be957eaa9c12c301`]([https://stellar.expert/explorer/testnet/tx/cfd18f9d08a3f213c6751ed372947657df2ef7f5b64a8524be957eaa9c12c301](https://stellar.expert/explorer/testnet/tx/cfd18f9d08a3f213c6751ed372947657df2ef7f5b64a8524be957eaa9c12c301))

- Evento `invest`: `[investor, 500, 5]`, es decir, 500 PAY ÷ 100 por unidad = 5 RWA

- Balance RWA del investor después de invertir: **5**

- PAY del investor: 1000 → 500

## Transacciones de la demo en video

### Admin tool (firma `alice`)

| Acción | Resultado | Transacción |

|---|---|---|

| `set_whitelist` (bob) | ✅ | `17649e40…`]([https://stellar.expert/explorer/testnet/tx/17649e40ccaceeffde5264819ac4c5e8dda3ed2087be8a2577f25e0c24de1dc9](https://stellar.expert/explorer/testnet/tx/17649e40ccaceeffde5264819ac4c5e8dda3ed2087be8a2577f25e0c24de1dc9)) |

| `mint` 100 RWA a bob | ✅ | `461282ac…`]([https://stellar.expert/explorer/testnet/tx/461282ac4d5e13ec5a9fbaf57d10ff92ea032fc130fa1bf4dc2adbab1c14b10e](https://stellar.expert/explorer/testnet/tx/461282ac4d5e13ec5a9fbaf57d10ff92ea032fc130fa1bf4dc2adbab1c14b10e)) |

| `withdraw` 500 PAY a la tesorería | ✅, tesorería = 1000 PAY | `ed381d7e…`]([https://stellar.expert/explorer/testnet/tx/ed381d7ed6d7131640a0a46df966a1b45273d7dbb17c6a0b904684beacc6d2c6](https://stellar.expert/explorer/testnet/tx/ed381d7ed6d7131640a0a46df966a1b45273d7dbb17c6a0b904684beacc6d2c6)) |

| `pause` | ✅ | `10c270e4…`]([https://stellar.expert/explorer/testnet/tx/10c270e4da54ba48c89f79a86c96da8decbeaf405c83d4379a36fc93baa37427](https://stellar.expert/explorer/testnet/tx/10c270e4da54ba48c89f79a86c96da8decbeaf405c83d4379a36fc93baa37427)) |

| `unpause` | ✅ | `a17e8501…`]([https://stellar.expert/explorer/testnet/tx/a17e85010b2114e8c65867c449bb2077eea4b585aa72fb8976bc072a63b7ddcd](https://stellar.expert/explorer/testnet/tx/a17e85010b2114e8c65867c449bb2077eea4b585aa72fb8976bc072a63b7ddcd)) |

### User tool (firma `bob`)

| Acción | Resultado | Transacción |

|---|---|---|

| `invest` 100 | ❌ `Error(Contract, #7)` AmountTooLow | (rechazada en simulación) |

| `invest` 500 | ✅, devuelve 5 RWA | `db6949fc…`]([https://stellar.expert/explorer/testnet/tx/db6949fc43797c12e6000f1affbc7a46a8ac260346f32a33fd25ad8a4023e403](https://stellar.expert/explorer/testnet/tx/db6949fc43797c12e6000f1affbc7a46a8ac260346f32a33fd25ad8a4023e403)) |

| `balance` | 215 RWA | (solo lectura) |

| `transfer` 10 RWA a la tesorería | ✅ | `a8c74859…`]([https://stellar.expert/explorer/testnet/tx/a8c74859c2e2438f1cd7d251cba1f0c08b6ec65b12093251db9644a5e67d6fa0](https://stellar.expert/explorer/testnet/tx/a8c74859c2e2438f1cd7d251cba1f0c08b6ec65b12093251db9644a5e67d6fa0)) |

## Cómo reproducir

### Requisitos

- Rust con target `wasm32v1-none`, y Stellar CLI

- Identidades con fondos en testnet: `alice` (admin), `bob` (investor) y `treasury`

```bash

stellar keys generate alice --network testnet --fund

stellar keys generate bob --network testnet --fund

stellar keys generate treasury --network testnet --fund

```

### Variables de entorno

Los scripts leen estas variables; no hace falta editarlos.

```bash

export NETWORK=testnet

export ADMIN_KEY=alice USER_KEY=bob

export CONTRACT_ID=CDUEDH4L3KDNIWAPWFRICMHYDEDMGIQ7P65U2GOPFJULOB4PVBUNC66X

export PAYMENT_TOKEN=CBJ343FZ6RYBSDPF6EDAOAIAK4C7NXOKSWZJALVTL5VJATMD4WEPHYRZ

export INVESTOR=$(stellar keys address bob)

export TREASURY=$(stellar keys address treasury)

export RECIPIENT=$(stellar keys address treasury)

```

### Deploy desde cero

```bash

cargo test

stellar contract build

stellar contract deploy --wasm target/wasm32v1-none/release/rwa_launchpad_dia_3.wasm --source alice --network testnet

```

Después ejecuta `initialize` y `set_whitelist` de `scripts/admin-tool.sh`. Si el contrato ya está desplegado e inicializado, pega los comandos del script de uno en uno. `initialize` solo puede ejecutarse una vez; si lo repites, falla con `#2` y el script se detiene por `set -e`.

### Probar la variación

```bash

# Falla con AmountTooLow (#7)

stellar contract invoke --id "$CONTRACT_ID" --source "$USER_KEY" --network "$NETWORK" -- invest --investor "$INVESTOR" --payment_amount 100

# Éxito

stellar contract invoke --id "$CONTRACT_ID" --source "$USER_KEY" --network "$NETWORK" -- invest --investor "$INVESTOR" --payment_amount 500

# Balance RWA

stellar contract invoke --id "$CONTRACT_ID" --source "$USER_KEY" --network "$NETWORK" -- balance --id "$INVESTOR"

```

## Notas

- **Stellar CLI 28:** los campos `i128` de `AssetInfo` deben ir como string en el JSON de `initialize` `"total_supply":"1000000","price_per_unit":"100"`). `scripts/admin-tool.sh` ya está actualizado.

- **Trustline:** una cuenta G necesita un trustline a `PAY` para recibir el token de pago: `stellar tx new change-trust --source <key> --line PAY:<ADMIN_ADDRESS> --network testnet`.

- **zsh:** para pegar comandos con comentarios `#` en la terminal, ejecuta antes `setopt interactivecomments`.

- **Orden de validaciones en `invest`:** autenticación, variación (mínimo 500), pausa, monto positivo y whitelist.