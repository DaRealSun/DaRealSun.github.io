---
title: 'Stage 2 tutorial: Laravel with the same contract'
---

## Repository and architecture

[Source](https://github.com/DaRealSun/campus-equipment-api/tree/main/code/stage2), [script notes](https://github.com/DaRealSun/campus-equipment-api/blob/main/code/stage2/SCRIPT-NOTES.md), [contract](https://github.com/DaRealSun/campus-equipment-api/blob/main/acceptance.md), [comparison](https://github.com/DaRealSun/campus-equipment-api/blob/main/presentation/comparison.md).

Stage 1 remains in `code/stage1`. Stage 2 uses Laravel routing/controllers and Eloquent models for users, equipment, loans and token digests. User has many loans/tokens. Equipment has many loans, including an active-loans relationship. Loan belongs to user and equipment. Application database operations use Eloquent; schema changes use Laravel migrations.

The contract is under `/api/v1`. Authentication uses equivalent opaque bearer tokens with 24-hour expiry. Registration always creates a borrower. Staff maintain equipment. Borrowers see their loans; staff see all loans. Only the owner or staff may return a loan. Loan history prevents equipment deletion.

## Start from prerequisites

Use Bash on Linux, WSL2 or macOS with Docker Engine/Compose, curl and OpenSSL. Docker must be running. For the CLI path, also install PHP 8.4 with PDO MySQL, Composer 2 and Python 3. No Node build is required. Commands start from the project repository root.

```sh
cd code/stage2
bash run.sh
```

This uses MySQL on localhost:33642 and serves Laravel at localhost:8641. Keep the terminal open. The script generates an ignored `.env` with random local secrets, installs locked dependencies and applies migrations. This machine's verification used a PHP CLI container wrapper with host networking; see the script notes for that limitation. It does not claim native Windows PHP.

For the complete container alternative:

```sh
cd code/stage2
bash setup.sh
curl --fail http://127.0.0.1:8642/up
```

Compose builds PHP-FPM, starts MySQL, and serves Laravel through NGINX at localhost:8642. Database readiness is checked before migrations. Startup does not reset data. Stop with `docker compose stop`; restart with `bash setup.sh`. Avoid port conflicts on 33642 and 8642.

## Load only synthetic acceptance fixtures

The following command deletes the synthetic Stage 2 records and recreates known fixtures. It requires explicit opt-in and the exact isolated database name. Do not point this setup at real data.

```sh
# From code/stage2, after setup.sh:
docker compose exec -T -e ALLOW_FIXTURE_RESET=1 php php artisan fixtures:reset
# CLI-path equivalent, in another terminal after run.sh:
ALLOW_FIXTURE_RESET=1 php artisan fixtures:reset
```

The fixture staff is ID 101. Borrowers are IDs 102/103. Tripod 1001 is available; camera 1002 has active loan 2001 owned by borrower 103. Synthetic fixture tokens are `fixture-staff-token`, `fixture-borrower1-token`, and `fixture-borrower2-token`. Fixture credentials are for this disposable local instance only, not production authentication.

## Ten endpoints

| Method and path | Access | Expected success and important failure |
|---|---|---|
| POST `/auth/register` | Public | 201 borrower; 400 invalid fields, 409 duplicate email |
| POST `/auth/login` | Public | 200 token/token_type; 401 wrong password |
| GET `/equipment` | Public | 200 list; 400 invalid available filter |
| GET `/equipment/{id}` | Public | 200 item; 404 missing |
| POST `/equipment` | Staff | 201 created item; 400 invalid name |
| PUT `/equipment/{id}` | Staff | 200 updated item; 404 missing |
| DELETE `/equipment/{id}` | Staff | 204 empty response; 409 loan history |
| POST `/loans` | Borrower | 201 new loan; 409 already on loan |
| GET `/loans` | Authenticated | 200 own loans or all for staff; 401 bad token |
| PUT `/loans/{id}/return` | Owner/staff | 200 returned loan; 403 other borrower, 409 repeated return |

Every path in the table has prefix `/api/v1`. All six protected endpoint types reject missing/invalid tokens with 401. Role and ownership denials use 403. Success uses `{"data": ...}` except an empty 204. Errors contain nonempty `error.code` and `error.message`. Names are 1–100 bytes, descriptions at most 1000 bytes, passwords 8–72 bytes on registration. IDs are positive integers. `available=true` or `false` filters the catalog. Details and all test cases are in the acceptance contract.

## Try a success and denied operation

```sh
BASE=http://127.0.0.1:8642/api/v1
curl -i "$BASE/equipment/1001"
# 200, data.id=1001, data.name="Tripod", data.available=true
curl -i -X POST "$BASE/auth/login" \
  -H 'Content-Type: application/json' \
  -d '{"email":"borrower1@example.test","password":"CourseFixturePassword-42"}'
# 200, data.token is a new opaque token, data.token_type="Bearer"
curl -i -X PUT "$BASE/loans/2001/return" \
  -H 'Authorization: Bearer fixture-borrower1-token'
# 403, error.code="forbidden", loan 2001 remains unchanged
curl -i "$BASE/loans"
# 401, error.code="authentication_required"
```

These statuses were observed by the shared acceptance/integrity checks. Do not paste real access tokens into reports. Loan records contain id, user_id, equipment_id, borrowed_at and nullable returned_at.

## Verify both implementations

Reset fixtures before each acceptance run. From the repository root:

```sh
python3 code/tests/acceptance.py --base-url http://127.0.0.1:8642 --stage stage2 --output docs/results/stage2-docker.json
API_BASE_URL=http://127.0.0.1:8642 python3 code/tests/stage2-integrity.py
```

For the CLI path, substitute 8641 and `stage2-host.json`. For Stage 1 use its documented startup/reset, port 8640 and `--stage stage1`. The assertions are unchanged. [Stage 1](https://github.com/DaRealSun/campus-equipment-api/blob/main/docs/results/stage1.json), [CLI](https://github.com/DaRealSun/campus-equipment-api/blob/main/docs/results/stage2-host.json) and [Docker](https://github.com/DaRealSun/campus-equipment-api/blob/main/docs/results/stage2-docker.json) reports each record 44/44 passing. Integrity checks additionally confirm one active loan under competing checkout and unchanged records after denied return.

Troubleshooting: use `docker compose ps` and `docker compose logs --tail=50 php nginx mysql`; check Docker is running, ports are free and generated `.env` remains local. Do not fix errors by disabling authentication or deleting data. The Docker build needs registry/package network access. GitHub Pages serves this tutorial, not the local API.

## Sources and limitations

Adapted from the [ASE230 Laravel and automation materials](https://github.com/nkuase/ase230/tree/main/module2), [Laravel routing](https://laravel.com/framework/docs/13.x/routing), [Eloquent](https://laravel.com/framework/docs/eloquent), [transactions](https://laravel.com/framework/docs/database), and [Docker Compose](https://docs.docker.com/compose/). OpenAI Codex drafted the project-specific code/tutorial and ran the recorded tests. No load test, security audit, production deployment or universal OS compatibility is claimed.
