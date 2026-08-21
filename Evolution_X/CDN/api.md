---
title: API Routes
---

# API Routes

Available routes for Evolution X CDN

## 1. Health Check
Method: `GET /api/health`

Authentication: `NULL`

Checks the system status for all components

Example response:
```json
{
  "database": {
    "status": "healthy",
    "message": "Database connection successful"
  },
  "filesystem": {
    "status": "healthy",
    "message": "Filesystem accessible",
    "base_path": "/mnt/evolution-x",
    "readable": true,
    "writable": true
  },
  "prerelease": {
    "status": "healthy",
    "message": "Pre-release path accessible",
    "path": "/mnt/pre-release",
    "exists": true,
    "readable": true
  },
  "r2": {
    "status": "healthy",
    "message": "R2 configuration present",
    "configured": true
  },
  "php_extensions": {
    "status": "healthy",
    "message": "All required extensions loaded",
    "required": ["pdo", "pdo_sqlite", "json", "curl"],
    "missing": []
  },
  "push_queue": {
    "status": "healthy",
    "message": "Push queue operational",
    "details": {
      "queued": 0,
      "processing": 0,
      "total_processed": 150
    }
  }
}
```

## 2. Download Statistics
Method: `GET /api/download-statistics`  
Alias: `GET /api/download-stats`

Authentication: `NULL`

Retreive download statistics with flexible filtering

Query Parameters:

| Parameter   | Type    | Required | Description                            | Example                                           |
|:------------|:--------|:--------:|:---------------------------------------|:--------------------------------------------------|
| `filename`  | string  | No       | Filter by filename pattern             | `EvolutionX-16.0-20260802-Pong-11.9-Official.zip` |
| `folder`    | string  | No       | Filter by folder path                  | `Pong/16`                                         |
| `timeStart` | string  | No       | Start Date (YYYY-MM-DD)                | `2026-02-28`                                      |
| `timeEnd`   | string  | No       | End Date (YYYY-MM-DD)                  | `2026-08-31`                                      |
| `limit`     | integer | No       | Result Limit (default: 50, max: 1000)  | `150`                                             |
| `sort`      | string  | No       | Sort by: `downloads`, `name`           | `downloads`                                       |
| `format`    | string  | No       | Response Format: `summary`, `detailed` | `summary`                                         |

Example response without parameters:
```json
[
  {
    "folder": "ALL_DOWNLOADS",
    "downloadCount": 2464813,
    "timeStart": "2025-09-13",
    "timeEnd": "2026-08-20",
    "individualFiles": [
      {
        "filename": "TOTAL_FILES_3133",
        "downloadCount": 2464813
      }
    ]
  }
]
```

Example response with below parameters:
- `folder` = `Pong/16`
- `timeStart` = `2026-08-02`
- `timeEnd` = `2026-08-16`

```json
[
  {
    "folder": "Pong/16",
    "downloadCount": 320,
    "timeStart": "2026-08-02",
    "timeEnd": "2026-08-16",
    "individualFiles": [
      {
        "filename": "EvolutionX-16.0-20260802-Pong-11.9-Official.zip",
        "downloadCount": 316
      },
      {
        "filename": "EvolutionX-16.0-20260713-Pong-11.9-Official.zip",
        "downloadCount": 4
      }
    ]
  },
  {
    "folder": "Pong/16_vanilla",
    "downloadCount": 62,
    "timeStart": "2026-08-02",
    "timeEnd": "2026-08-16",
    "individualFiles": [
      {
        "filename": "EvolutionX-16.0-20260802-Pong-11.9-Vanilla-Official.zip",
        "downloadCount": 59
      },
      {
        "filename": "EvolutionX-16.0-20260618-Pong-11.8-Vanilla-Official.zip",
        "downloadCount": 3
      }
    ]
  },
  {
    "folder": "Pong/16/recovery",
    "downloadCount": 58,
    "timeStart": "2026-08-02",
    "timeEnd": "2026-08-16",
    "individualFiles": [
      {
        "filename": "recovery.img",
        "downloadCount": 58
      }
    ]
  },
  {
    "folder": "Pong/16_vanilla/recovery",
    "downloadCount": 43,
    "timeStart": "2026-08-02",
    "timeEnd": "2026-08-16",
    "individualFiles": [
      {
        "filename": "recovery.img",
        "downloadCount": 43
      }
    ]
  }
]
```

## 3. File Hashes
Method `GET /api/hash`

Authentication: `NULL`

Get file checksums (md5, sha256)

Query Parameters:

| Parameter | Type   | Required | Description                     | Example                                                   |
|:----------|:-------|:--------:|:--------------------------------|:----------------------------------------------------------|
| `path`    | string | Yes      | Full file path including folder | `Pong/16/EvolutionX-16.0-20260802-Pong-11.9-Official.zip` |

Example response with below parameters:
- `path` = `Pong/16/EvolutionX-16.0-20260802-Pong-11.9-Official.zip`

```json
{
  "success": true,
  "path": "Pong/16/EvolutionX-16.0-20260802-Pong-11.9-Official.zip",
  "recursive": false,
  "files": [
    {
      "key": "Pong/16/EvolutionX-16.0-20260802-Pong-11.9-Official.zip",
      "sha256": "6bf18bb13103464919c98b6721e5fdb42ac8f6a45645f5b3fb98f821ed9789be",
      "md5": "c03a78845c94c05027b93cf563cb3a96",
      "file_size": 3631730909,
      "count": 359
    }
  ]
}
```

## 4. Push Release
Method: `POST /api/push`

Authentication: Internal useragent and token

Release builds from one storage bucket to another.

Headers:
```
Content-Type: application/json
```

Request Body:
```json
{
    "codename": "Pong",
    "date": 20260802,
    "version": "11.9",
    "buildType": "GAPPS"
}
```

Post Fields:

| Field       | Type   | Required | Description                       | Example    |
|:------------|:-------|:--------:|:----------------------------------|:-----------|
| `codename`  | string | Yes      | Device Codename                   | `Pong`     |
| `date`      | string | Yes      | Build Date (YYYYMMDD)             | `20260802` |
| `version`   | string | Yes      | Evolution X Version               | `11.9`     |
| `buildType` | string | Yes      | Build Type (`gapps` or `vanilla`) | `gapps`    |

Accepted response:
```json
{
    "status": "success",
    "APICode": "T-0007",
    "message": "Push release queued successfully",
    "jobId": 42,
    "queuePosition": 6
}
```

Rejected response:
```json
{
    "status": "error",
    "APICode": "T-0003",
    "message": "Authentication failed"
}
```

Example:
```sh
curl -X POST "https://cdn.evolution-x.org/api/push" \
  -H "Authorization: Bearer INTERNALPASSWORD" \
  -H "User-Agent: EvoXUserAgent/67" \
  -H "Content-Type: application/json" \
  -d '{
    "codename": "Pong",
    "date": 20260802,
    "version": "11.9",
    "buildType: "gapps"
  }'
```

#### Note: 
At the time a job is either marked as `completed` or `failed`, a Discord webhook message is sent to alert either the readiness of the files for OTA to be processed, or an alert is sent with an urgent tag to bring alarm to the error.

## 5. List Push Release Jobs
Method: `GET /api/push/jobs`

Authentication: Internal useragent and token

List push release jobs in the queue

Query Parameters:

| Parameter | Type    | Required | Description                           | Values                                        |
|:----------|:--------|:--------:|:--------------------------------------|:----------------------------------------------|
| `status`  | string  | No       | Filter by job status                  | `queued`, `processing`, `completed`, `failed` |
| `limit`   | integer | No       | Result Limit (default: 100, max: 500) | `50`                                          |

Example response:
```json
{
    "success": true,
    "data": {
        "jobs" [
            {
                "id": 42,
                "codename": "Pong",
                "release_date": "2026-08-02",
                "version": "11.9",
                "build_type": "gapps",
                "requested_by": "INTERNAL_USERAGENT",
                "source_path": "/mnt/pre-release/Pong/2026-08-02/",
                "destination_path": "/Pong/16/",
                "status": "completed",
                "success": true,
                "error_message": null,
                "callback_enabled": true,
                "callback_url": "https://example.com/callbackurl...",
                "callback_http_code": 200,
                "callback_response": "OK",
                "created_at": "2026-08-02 12:25:58",
                "started_at": "2026-08-02 12:27:01",
                "completed_at": "2026-08-02 12:29:24",
                "updated_at": "2026-08-02 12:29:24"
            }
        ],
        "count": 1
    }
}
```

Example request:
```sh
curl "https://cdn.evolution-x.org/api/push/jobs" \
  -H "Authorization: Bearer INTERNALPASSWORD" \
  -H "User-Agent: EvoXUserAgent/67"
```

## 6. Get Push Release Job Details
Method: `GET /api/push/jobs/{id}`

Authentication: Internal useragent and token

Get detailed job information for a specific release

Example Response:
```json
{
    "success": true,
    "data": {
        "job": {
            "id": 42,
            "codename": "Pong",
            "release_date": "2026-08-02",
            "version": "11.9",
            "build_type": "gapps",
            "status": "completed",
            "success": true,
            "created_at": "2026-08-02 12:25:58",
            "completed_at": "2026-08-02 12:29:24"
        }
    }
}
```

Example request:
```sh
curl "https://cdn.evolution-x.org/api/push/jobs/42" \
  -H "Authorization: Bearer INTERNALPASSWORD" \
  -H "User-Agent: EvoXUserAgent/67"
```

## 7. Get Push Release Queue Statistics
Method: `GET /api/push/stats`

Authentication: Internal useragent and token

Get statistics about the push release queue

Example Response:
```json
{
  "success": true,
  "data": {
    "queued": 3,
    "processing": 1,
    "completed_today": 15,
    "completed_this_week": 87,
    "completed_total": 1523,
    "failed_today": 0,
    "failed_this_week": 2,
    "failed_total": 45,
    "average_processing_time": "4m 32s",
    "oldest_queued_job": {
      "id": 40,
      "created_at": "2026-08-02 11:07:15",
      "waiting_time": "1h 5m"
    }
  }
}

Example request:
```sh
curl "https://cdn.evolution-x.org/api/push/stats" \
  -H "Authorization: Bearer INTERNALPASSWORD" \
  -H "User-Agent: EvoXUserAgent/67"
```

## 8. Daily Downloads
Method: `GET /api/daily-downloads`

Authentication: `NULL`

Get daily totals for the given date.  Defaults to previous day.

Query Parameters:

| Parameter | Type   | Required | Description                            |
|:----------|:-------|:--------:|:---------------------------------------|
| `date`    | string | No       | Specify the date you want (YYYY-MM-DD) |

Example Response:
```json
{
    "fordate": "2026-08-19",
    "summary": {
        "total_downloads": 3031,
        "unique_files_downloaded": 509,
        "total_download_events": 3031
    },
    "individual_files": [
        {
            "filename": "spes/16/EvolutionX-16.0-20260807-spes-11.10-Official.zip",
            "downloads": 137
        },
        {
            "filename": "raphael/17/EvolutionX-17.0-20260812-raphael-12.1-Official.zip",
            "downloads": 91
        },
        {
            "filename": "marble/17/EvolutionX-17.0-20260812-marble-12.1-Official.zip",
            "downloads": 83
        },
        {
            "filename": "dodge/16/EvolutionX-16.0-20260816-dodge-11.10-Official.zip",
            "downloads": 67
        }
    ]
}
```

Example request:
```sh
curl "https://cdn.evolution-x.org/api/daily-downloads/2026-08-02"
```

## 9. Daily Summary
Method: `GET /api/daily-summary`

Authentication: `NULL`

Get daily totals for the last given timeframe.

Query Parameters:

| Parameter | Type    | Required | Description                                |
|:----------|:--------|:--------:|:-------------------------------------------|
| `days`    | integer | No       | Number of days to summarize.  Default is 7 |

Example Response:
```json
{
  "period": "7 days",
  "daily_summary": [
    {
      "date": "2026-08-21",
      "unique_files": 283,
      "total_downloads": 793
    },
    {
      "date": "2026-08-20",
      "unique_files": 506,
      "total_downloads": 2899
    },
    {
      "date": "2026-08-19",
      "unique_files": 509,
      "total_downloads": 3031
    },
    {
      "date": "2026-08-18",
      "unique_files": 502,
      "total_downloads": 3110
    },
    {
      "date": "2026-08-17",
      "unique_files": 514,
      "total_downloads": 3273
    },
    {
      "date": "2026-08-16",
      "unique_files": 558,
      "total_downloads": 4088
    },
    {
      "date": "2026-08-15",
      "unique_files": 536,
      "total_downloads": 3899
    },
    {
      "date": "2026-08-14",
      "unique_files": 583,
      "total_downloads": 4225
    }
  ]
}
```

Example request:
```sh
curl "https://cdn.evolution-x.org/api/daily-summary/14"
```

## 10. Stats Dashboard
Method: `GET /api/stats-dashboard`

Authentication: `NULL`

Most of the stats used on the /stats information page

Query Parameters:

| Parameter            | Type    | Required | Description                         | Default | Available Options                                                                        |
|:---------------------|:--------|:--------:|:------------------------------------|:--------|:-----------------------------------------------------------------------------------------|
| `breakdownTimeframe` | string  | No       | Timeframe for the downloads chart   | `7d`    | `today`, `7d`, `30d`, `all`                                                              |
| `devicesTimeframe`   | string  | No       | Timeframe for the top-devices table | `7d`    | `today`, `7d`, `30d`, `all`                                                              |
| `device`             | string  | No       | Codename filter for top-devices     | _(all)_ | _any letters, numbers and `.`, `_`, `-` only. Empty string or `all` returns all devices_ |
| `topLimit`           | integer | No       | Max rows in top-devices table       | `25`    | 1-100                                                                                    |

Example response:
```json
{
  "success": true,
  "has_data": true,
  "table_available": true,
  "cache_source": "cron_json",
  "cache_generated_at": "2026-08-21T07:30:42+00:00",
  "filters": {
    "breakdown_timeframe": "7d",
    "devices_timeframe": "7d",
    "device": "all",
    "top_limit": 25
  },
  "summary": {
    "total_downloads": 2469850,
    "since_date": "2025-09-13"
  },
  "download_breakdown": {
    "timeframe": "7d",
    "series": [
      {
        "date": "2026-08-15",
        "day": "Sat",
        "downloads": 3899
      },
      {
        "date": "2026-08-16",
        "day": "Sun",
        "downloads": 4088
      },
      {
        "date": "2026-08-17",
        "day": "Mon",
        "downloads": 3273
      },
      {
        "date": "2026-08-18",
        "day": "Tue",
        "downloads": 3110
      },
      {
        "date": "2026-08-19",
        "day": "Wed",
        "downloads": 3031
      },
      {
        "date": "2026-08-20",
        "day": "Thu",
        "downloads": 2899
      },
      {
        "date": "2026-08-21",
        "day": "Fri",
        "downloads": 797
      }
    ]
  },
  "top_devices": {
    "timeframe": "7d",
    "rows": [
      {
        "device": "marble",
        "display_name": "Xiaomi Poco F5 / Redmi Note 12 Turbo (marble)",
        "downloads": 1800
      },
      {
        "device": "raphael",
        "display_name": "Xiaomi Mi 9T Pro / Redmi K20 Pro (raphael)",
        "downloads": 1670
      },
      {
        "device": "spes",
        "display_name": "Xiaomi Redmi Note 11 (spes)",
        "downloads": 1134
      },
      {
        "device": "sweet",
        "display_name": "Xiaomi Redmi Note 10 Pro / Max (sweet)",
        "downloads": 1028
      },
      {
        "device": "redwood",
        "display_name": "Xiaomi Poco X5 Pro / Redmi Note 12 Pro Speed (redwood)",
        "downloads": 733
      },
      {
        "device": "venus",
        "display_name": "Xiaomi Mi 11 (venus)",
        "downloads": 494
      },
      {
        "device": "miatoll",
        "display_name": "Xiaomi Poco M2 Pro / Redmi Note 9S / Note 9 Pro / Note 9 Pro Max / Note 10 Lite (miatoll)",
        "downloads": 457
      },
      {
        "device": "tissot",
        "display_name": "Xiaomi Mi A1 (tissot)",
        "downloads": 427
      },
      {
        "device": "garnet",
        "display_name": "Xiaomi Redmi Note 13 Pro 5G / Poco X6 5G (garnet)",
        "downloads": 420
      },
      {
        "device": "ginkgo",
        "display_name": "Xiaomi Redmi Note 8 / Note 8T (ginkgo)",
        "downloads": 389
      },
      {
        "device": "veux",
        "display_name": "Xiaomi Redmi Note 11 Pro Plus 5G / Poco X4 Pro 5G (veux)",
        "downloads": 380
      },
      {
        "device": "d2s",
        "display_name": "Samsung Galaxy Note 10 Plus (d2s)",
        "downloads": 335
      },
      {
        "device": "dodge",
        "display_name": "OnePlus 13 (dodge)",
        "downloads": 330
      },
      {
        "device": "panther",
        "display_name": "Google Pixel 7 (panther)",
        "downloads": 325
      },
      {
        "device": "duchamp",
        "display_name": "Xiaomi POCO X6 Pro 5G / Redmi K70E (duchamp)",
        "downloads": 303
      },
      {
        "device": "beyond2lte",
        "display_name": "Samsung Galaxy S10 Plus (beyond2lte)",
        "downloads": 302
      },
      {
        "device": "cheetah",
        "display_name": "Google Pixel 7 Pro (cheetah)",
        "downloads": 298
      },
      {
        "device": "a52sxq",
        "display_name": "Samsung Galaxy A52s 5G (a52sxq)",
        "downloads": 291
      },
      {
        "device": "oriole",
        "display_name": "Google Pixel 6 (oriole)",
        "downloads": 260
      },
      {
        "device": "cepheus",
        "display_name": "Xiaomi Mi 9 (cepheus)",
        "downloads": 252
      },
      {
        "device": "vayu",
        "display_name": "Xiaomi Poco X3 Pro (vayu)",
        "downloads": 252
      },
      {
        "device": "dm3q",
        "display_name": "Samsung Galaxy S23 Ultra (dm3q)",
        "downloads": 241
      },
      {
        "device": "sapphire",
        "display_name": "Xiaomi Redmi Note 13 4G/NFC (sapphire)",
        "downloads": 229
      },
      {
        "device": "kebab",
        "display_name": "OnePlus 8T (kebab)",
        "downloads": 221
      },
      {
        "device": "instantnoodle",
        "display_name": "OnePlus 8 (instantnoodle)",
        "downloads": 217
      }
    ],
    "top_device": {
      "device": "marble",
      "display_name": "Xiaomi Poco F5 / Redmi Note 12 Turbo (marble)",
      "downloads": 1800
    }
  },
  "devices_available": [
    "a52sxq",
    "a71",
    "akita",
    "alioth",
    "apollon",
    "aston",
    "bangkk",
    "bathena",
    "beckham",
    "benz",
    "berlna",
    "beryllium",
    "beyond0lte",
    "beyond1lte",
    "beyond2lte",
    "beyondx",
    "blazer",
    "bluejay",
    "blueline",
    "bonito",
    "borneo",
    "bramble",
    "caiman",
    "capri",
    "caprip",
    "cebu",
    "cepheus",
    "channel",
    "cheeseburger",
    "cheetah",
    "cmi",
    "comet",
    "crosshatch",
    "cupid",
    "d1",
    "d2s",
    "d2x",
    "davinci",
    "devon",
    "dm1q",
    "dm2q",
    "dm3q",
    "dodge",
    "dubai",
    "duchamp",
    "dumpling",
    "evert",
    "f62",
    "felix",
    "fleur",
    "frankel",
    "garnet",
    "ginkgo",
    "gio",
    "gta4xl",
    "gta4xlwifi",
    "gts4lv",
    "gts4lvwifi",
    "guacamole",
    "guacamoleb",
    "guam",
    "guamp",
    "hawao",
    "haydn",
    "hotdog",
    "hotdogb",
    "husky",
    "instantnoodle",
    "instantnoodlep",
    "kane",
    "kebab",
    "komodo",
    "lake",
    "laurel_sprout",
    "lemonade",
    "lemonadep",
    "lemonades",
    "LH7n",
    "lisa",
    "lynx",
    "marble",
    "miatoll",
    "mido",
    "mondrian",
    "monet",
    "mumba",
    "mustang",
    "nemo",
    "nio",
    "ocean",
    "op6893",
    "oriole",
    "panther",
    "payton",
    "penang",
    "peridot",
    "PL2",
    "pl2",
    "polaris",
    "Pong",
    "pstar",
    "pyxis",
    "r8s",
    "rango",
    "raphael",
    "raven",
    "redfin",
    "redwood",
    "rhode",
    "rhodep",
    "river",
    "RMX2061",
    "rmx2061",
    "rosemary",
    "rtwo",
    "rubyx",
    "salaa",
    "salami",
    "sapphire",
    "sargo",
    "scorpio",
    "shiba",
    "sky",
    "Spacewar",
    "spartan",
    "spes",
    "stone",
    "sunfish",
    "sweet",
    "tangorpro",
    "taoyao",
    "tegu",
    "Tetris",
    "timelm",
    "tissot",
    "tokay",
    "topaz",
    "troika",
    "tundra",
    "umi",
    "vayu",
    "venus",
    "veux",
    "vince",
    "violet",
    "xpeng",
    "zeus",
    "zippo"
  ],
  "selected_device": {
    "device": "all",
    "downloads": 0,
    "exists": false
  }
}
```

Example request:
```sh
curl "https://cdn.evolution-x.org/api/stats-dashboard?devicesTimeframe=30d&device=Pong"
```

