# SCAE Backend — Complete Build Prompts
# ══════════════════════════════════════════════════════════
# 7 prompts total. Run in ORDER. Never skip one.
# Goal: Full Java HTTP Server + C Algorithm Engine + JSON File Storage
# Stack: Java (com.sun.net.httpserver) | C (JNI) | JSON Files | JWT Auth
# Postman File: SCAE_Backend_API_Collection.json (55 endpoints, 10 folders)
# Test Script:  scae_test_all.sh  → bash scae_test_all.sh
# Data verify:  scae_data_test.sh → bash scae_data_test.sh
# Deploy:       Render.com → render.yaml
# ══════════════════════════════════════════════════════════

---

## ★★★ MASTER CONTEXT — Paste this at the TOP of EVERY prompt ★★★

```
PROJECT: SCAE — Smart City Algorithm Engine
STACK: Java (com.sun.net.httpserver) + C (JNI) + JSON Files + JWT Auth
FRONTEND: https://bucolic-biscotti-dfb4d5.netlify.app (Netlify — HTML/CSS/JS)
BACKEND DEPLOY: Render.com (render.yaml)

RESPONSE FORMAT (every API must return this):
  Success → { "success": true,  "message": "...", "data": { ... } }
  Error   → { "success": false, "message": "...", "error": "..."  }

COMMENT STYLE — ALL code must use Hinglish comments like this:
  // server create kiya gaya hai port 8000 pe
  // ye handler request ko receive karega
  // JSON file se data read kar rahe hai
  // algorithm ka result return ho raha hai

KEY RULES:
1. EVERY admin/dept route MUST use AuthMiddleware.java (JWT check)
2. ALL data stored in JSON files inside data/ folder — NO database
3. ALL algorithm logic MUST be in C files inside algorithms/ folder
4. Java calls C algorithms via JNI (Java Native Interface)
5. CORS must allow: https://bucolic-biscotti-dfb4d5.netlify.app
6. JWT token expires in 8 hours (HS256 signing)
7. Passwords stored as bcrypt hash in users.json
8. Port: 8000 (Render uses PORT env variable — always read from env)

FOLDER STRUCTURE:
scae-backend/
├── ScaeServer.java          ← main server — port 8000, all routes mount hote hai
├── render.yaml              ← Render deploy config
├── Makefile                 ← C compile karne ke liye
├── .env.example
├── handlers/
│   ├── AuthHandler.java     → /api/auth/*
│   ├── GraphHandler.java    → /api/graph/*
│   ├── AlgoHandler.java     → /api/algo/*
│   ├── ComplaintHandler.java → /api/complaints/*
│   ├── WorkOrderHandler.java → /api/work-orders/*
│   ├── EmergencyHandler.java → /api/emergencies/*
│   ├── CitizenHandler.java  → /api/citizens/*
│   ├── ProjectHandler.java  → /api/projects/*
│   ├── NoticeHandler.java   → /api/notices/*
│   ├── ServiceHandler.java  → /api/services/*
│   └── StatsHandler.java    → /api/stats/*
├── middleware/
│   └── AuthMiddleware.java  → JWT verify + role check
├── utils/
│   ├── JsonUtil.java        → JSON file read/write helper
│   ├── JwtUtil.java         → JWT generate + verify
│   └── CorsUtil.java        → CORS headers add karne ke liye
├── algorithms/              ← saare C files yahan hai
│   ├── scae.h               → shared structs (Graph, Edge, AlgoResult)
│   ├── dijkstra.c
│   ├── bellman_ford.c
│   ├── floyd_warshall.c
│   ├── bfs.c
│   ├── dfs.c
│   ├── kruskal.c
│   ├── prim.c
│   ├── knapsack_01.c
│   ├── knapsack_frac.c
│   ├── huffman.c
│   ├── kmp.c
│   ├── rabin_karp.c
│   ├── tsp_greedy.c
│   ├── tsp_brute.c
│   ├── job_seq.c
│   ├── activity_sel.c
│   ├── heap_sort.c
│   ├── lcs.c
│   ├── benchmark.c
│   └── ScaeJNI.c            → JNI wrapper — Java se C call hoti hai yahan
└── data/                    ← saare JSON data files yahan hai
    ├── users.json
    ├── complaints.json
    ├── work_orders.json
    ├── emergencies.json
    ├── nodes.json
    ├── edges.json
    ├── notices.json
    ├── projects.json
    ├── services.json
    └── sensor_logs.json

JSON DATA FILES — EXPECTED FINAL STATE (after all 7 prompts):
  users.json        → 3 users (admin, citizen, dept)
  nodes.json        → 13 city nodes (Central Junction to Sports Complex)
  edges.json        → 19 city edges (with weights)
  complaints.json   → 3 sample complaints
  work_orders.json  → 4 sample work orders
  emergencies.json  → 5 sample emergencies
  notices.json      → 3 public notices
  projects.json     → 6 budget projects (for Knapsack)
  services.json     → 10 city services (hospital, school, etc.)
  sensor_logs.json  → 3 sensor readings (for Huffman)
```

---
---

# ══════════════════════════════════════════════════════════
# PROMPT 1 — Project Setup + Server + Health + CORS + C Compile
# ══════════════════════════════════════════════════════════
# Test after this → GET /api/health → { success: true, status: "ok" }
# Data verify     → ls data/ → 10 JSON files visible (empty arrays)
# C verify        → make → libscae.so/scae.dll compiles without error

[Paste MASTER CONTEXT above this line]

You are a senior Java + C backend developer building the SCAE backend.
Use Hinglish comments in ALL code (see MASTER CONTEXT for style).

TASK: Project folder setup + ScaeServer.java + health endpoint + CORS + Makefile + empty data files.

## STEP A — Create folder structure

```bash
mkdir scae-backend && cd scae-backend
mkdir handlers middleware utils algorithms data

# Java ke liye koi npm nahi — sirf javac use hoga
# C compile karne ke liye GCC chahiye
```

Create ALL files listed in FOLDER STRUCTURE as empty placeholders first,
then fill them one by one as described below.

## STEP B — Write ScaeServer.java

```java
// ScaeServer.java — SCAE ka main entry point
// ye file server start karti hai aur saare routes mount karti hai

import com.sun.net.httpserver.HttpServer;
import java.net.InetSocketAddress;

public class ScaeServer {

    public static void main(String[] args) throws Exception {

        // PORT environment variable se port read karo — Render ke liye zaroori hai
        int port = Integer.parseInt(System.getenv().getOrDefault("PORT", "8000"));

        // server create kiya gaya hai given port pe
        HttpServer server = HttpServer.create(new InetSocketAddress(port), 0);

        // health check endpoint — sabse pehle test hota hai
        server.createContext("/api/health", new handlers.HealthHandler());

        // auth routes — login, profile
        server.createContext("/api/auth", new handlers.AuthHandler());

        // graph routes — nodes, edges
        server.createContext("/api/graph", new handlers.GraphHandler());

        // algorithm routes — dijkstra, knapsack, etc.
        server.createContext("/api/algo", new handlers.AlgoHandler());

        // complaints routes
        server.createContext("/api/complaints", new handlers.ComplaintHandler());

        // work orders routes
        server.createContext("/api/work-orders", new handlers.WorkOrderHandler());

        // emergencies routes
        server.createContext("/api/emergencies", new handlers.EmergencyHandler());

        // citizens routes — admin only
        server.createContext("/api/citizens", new handlers.CitizenHandler());

        // projects routes — budget knapsack ke liye
        server.createContext("/api/projects", new handlers.ProjectHandler());

        // notices routes — public announcements
        server.createContext("/api/notices", new handlers.NoticeHandler());

        // services routes — city services
        server.createContext("/api/services", new handlers.ServiceHandler());

        // stats routes — dashboard counters
        server.createContext("/api/stats", new handlers.StatsHandler());

        // default executor use ho raha hai
        server.setExecutor(null);

        // server start kar diya
        server.start();
        System.out.println("✅ SCAE Server started at http://localhost:" + port);
        System.out.println("📊 Algorithm Engine: C algorithms via JNI");
        System.out.println("💾 Storage: JSON files in data/ folder");
    }
}
```

## STEP C — Write handlers/HealthHandler.java

```java
// HealthHandler.java — server ka health check endpoint
// GET /api/health → server chalra hai ya nahi check karta hai

package handlers;

import com.sun.net.httpserver.HttpExchange;
import com.sun.net.httpserver.HttpHandler;
import utils.CorsUtil;
import java.io.*;

public class HealthHandler implements HttpHandler {

    public void handle(HttpExchange exchange) throws IOException {

        // CORS headers add karo — frontend se request aayegi
        CorsUtil.addCorsHeaders(exchange);

        // OPTIONS request handle karo (preflight)
        if ("OPTIONS".equals(exchange.getRequestMethod())) {
            exchange.sendResponseHeaders(204, -1);
            return;
        }

        // health response banao
        String response = "{"
            + "\"success\": true,"
            + "\"message\": \"SCAE Server is running\","
            + "\"data\": {"
            +   "\"status\": \"ok\","
            +   "\"server\": \"SCAE Java Backend\","
            +   "\"algorithms\": \"C via JNI\","
            +   "\"storage\": \"JSON files\","
            +   "\"timestamp\": \"" + new java.util.Date() + "\""
            + "}"
            + "}";

        // response bhejo
        exchange.getResponseHeaders().set("Content-Type", "application/json");
        exchange.sendResponseHeaders(200, response.getBytes().length);
        OutputStream os = exchange.getResponseBody();
        os.write(response.getBytes());
        os.close();
    }
}
```

## STEP D — Write utils/CorsUtil.java

```java
// CorsUtil.java — CORS headers add karne ka helper
// frontend Netlify pe hai — usse allow karna zaroori hai

package utils;

import com.sun.net.httpserver.HttpExchange;

public class CorsUtil {

    // frontend ka URL — ye Netlify pe hai
    private static final String FRONTEND_URL =
        System.getenv().getOrDefault("FRONTEND_URL",
        "https://bucolic-biscotti-dfb4d5.netlify.app");

    public static void addCorsHeaders(HttpExchange exchange) {

        // frontend URL ko allow karo
        exchange.getResponseHeaders().set("Access-Control-Allow-Origin", FRONTEND_URL);

        // ye methods allow hai
        exchange.getResponseHeaders().set("Access-Control-Allow-Methods",
            "GET, POST, PUT, PATCH, DELETE, OPTIONS");

        // ye headers allow hai — Authorization JWT ke liye chahiye
        exchange.getResponseHeaders().set("Access-Control-Allow-Headers",
            "Content-Type, Authorization");

        // response cache ho sakta hai 1 ghante ke liye
        exchange.getResponseHeaders().set("Access-Control-Max-Age", "3600");
    }
}
```

## STEP E — Write utils/JsonUtil.java

```java
// JsonUtil.java — JSON files read/write karne ka helper
// ye class data/ folder ke files se data padhti aur likhti hai

package utils;

import java.io.*;
import java.nio.file.*;

public class JsonUtil {

    // data folder ka path — yahan saare JSON files hain
    private static final String DATA_DIR = "data/";

    // file se JSON string read karo
    public static String readFile(String filename) throws IOException {
        Path path = Paths.get(DATA_DIR + filename);
        // agar file nahi hai toh empty array return karo
        if (!Files.exists(path)) {
            return "[]";
        }
        return new String(Files.readAllBytes(path));
    }

    // JSON string file mein write karo
    public static void writeFile(String filename, String jsonContent) throws IOException {
        Path path = Paths.get(DATA_DIR + filename);
        // parent directory create karo agar nahi hai
        Files.createDirectories(path.getParent());
        Files.write(path, jsonContent.getBytes());
    }
}
```

## STEP F — Write utils/JwtUtil.java

```java
// JwtUtil.java — JWT token generate aur verify karne ka helper
// JWT library nahi hai toh manual HS256 implementation karenge

package utils;

import javax.crypto.Mac;
import javax.crypto.spec.SecretKeySpec;
import java.util.Base64;
import java.util.Date;

public class JwtUtil {

    // JWT secret .env se aata hai
    private static final String SECRET =
        System.getenv().getOrDefault("JWT_SECRET", "scae-default-secret-key-32chars!!");

    // token 8 ghante ke liye valid hai
    private static final long EXPIRY_MS = 8 * 60 * 60 * 1000;

    // JWT token generate karo — login ke baad milta hai
    public static String generateToken(String email, String role, String name) {
        try {
            // header banao
            String header = Base64.getUrlEncoder().withoutPadding()
                .encodeToString("{\"alg\":\"HS256\",\"typ\":\"JWT\"}".getBytes());

            // payload banao — role aur name include karo
            long exp = System.currentTimeMillis() + EXPIRY_MS;
            String payloadJson = "{"
                + "\"sub\":\"" + email + "\","
                + "\"role\":\"" + role + "\","
                + "\"name\":\"" + name + "\","
                + "\"exp\":" + exp
                + "}";
            String payload = Base64.getUrlEncoder().withoutPadding()
                .encodeToString(payloadJson.getBytes());

            // signature banao — HS256 use hoga
            String dataToSign = header + "." + payload;
            Mac mac = Mac.getInstance("HmacSHA256");
            mac.init(new SecretKeySpec(SECRET.getBytes(), "HmacSHA256"));
            String signature = Base64.getUrlEncoder().withoutPadding()
                .encodeToString(mac.doFinal(dataToSign.getBytes()));

            // final token return karo
            return header + "." + payload + "." + signature;
        } catch (Exception e) {
            throw new RuntimeException("Token generate nahi ho saka: " + e.getMessage());
        }
    }

    // token verify karo — middleware mein use hota hai
    // returns payload JSON string ya null if invalid
    public static String verifyToken(String token) {
        try {
            String[] parts = token.split("\\.");
            if (parts.length != 3) return null;

            // signature verify karo
            String dataToSign = parts[0] + "." + parts[1];
            Mac mac = Mac.getInstance("HmacSHA256");
            mac.init(new SecretKeySpec(SECRET.getBytes(), "HmacSHA256"));
            String expectedSig = Base64.getUrlEncoder().withoutPadding()
                .encodeToString(mac.doFinal(dataToSign.getBytes()));

            if (!expectedSig.equals(parts[2])) return null; // signature match nahi hua

            // payload decode karo
            String payload = new String(Base64.getUrlDecoder().decode(parts[1]));

            // expiry check karo
            int expIdx = payload.indexOf("\"exp\":");
            if (expIdx != -1) {
                long exp = Long.parseLong(payload.substring(expIdx + 6,
                    payload.indexOf("}", expIdx + 6) == -1
                    ? payload.indexOf(",", expIdx + 6)
                    : Math.min(payload.indexOf("}", expIdx + 6), payload.indexOf(",", expIdx + 6))));
                if (System.currentTimeMillis() > exp) return null; // token expire ho gaya
            }

            return payload;
        } catch (Exception e) {
            return null; // invalid token
        }
    }
}
```

## STEP G — Write middleware/AuthMiddleware.java

```java
// AuthMiddleware.java — JWT check karta hai
// ye class verify karti hai ki request authorized hai ya nahi

package middleware;

import utils.JwtUtil;
import com.sun.net.httpserver.HttpExchange;

public class AuthMiddleware {

    // token verify karo aur payload return karo
    // null return matlab unauthorized
    public static String verify(HttpExchange exchange) {

        // Authorization header read karo
        String authHeader = exchange.getRequestHeaders().getFirst("Authorization");

        // header nahi hai
        if (authHeader == null || !authHeader.startsWith("Bearer ")) {
            return null;
        }

        // "Bearer " remove karo — sirf token chahiye
        String token = authHeader.substring(7);

        // token verify karo
        return JwtUtil.verifyToken(token);
    }

    // role check karo — admin only routes ke liye
    public static boolean isAdmin(String payload) {
        return payload != null && payload.contains("\"role\":\"admin\"");
    }

    // dept ya admin — work orders ke liye
    public static boolean isDeptOrAdmin(String payload) {
        return payload != null &&
            (payload.contains("\"role\":\"admin\"") || payload.contains("\"role\":\"dept\""));
    }

    // error response bhejo — unauthorized
    public static void sendUnauthorized(HttpExchange exchange) throws java.io.IOException {
        String body = "{\"success\":false,\"message\":\"Unauthorized — token required\"}";
        exchange.getResponseHeaders().set("Content-Type", "application/json");
        exchange.sendResponseHeaders(401, body.getBytes().length);
        java.io.OutputStream os = exchange.getResponseBody();
        os.write(body.getBytes());
        os.close();
    }

    // error response bhejo — forbidden (role nahi hai)
    public static void sendForbidden(HttpExchange exchange) throws java.io.IOException {
        String body = "{\"success\":false,\"message\":\"Forbidden — admin role required\"}";
        exchange.getResponseHeaders().set("Content-Type", "application/json");
        exchange.sendResponseHeaders(403, body.getBytes().length);
        java.io.OutputStream os = exchange.getResponseBody();
        os.write(body.getBytes());
        os.close();
    }
}
```

## STEP H — Write Makefile (C compile karne ke liye)

```makefile
# Makefile — C algorithm files compile karne ke liye
# make → libscae.so banata hai (Linux/Render)
# make windows → scae.dll banata hai (Windows)

CC = gcc
CFLAGS = -O2 -Wall -fPIC
ALGOS_DIR = algorithms
JNI_INCLUDE = -I$(JAVA_HOME)/include -I$(JAVA_HOME)/include/linux

# saare C files ko ek shared library mein compile karo
SRCS = $(ALGOS_DIR)/ScaeJNI.c \
       $(ALGOS_DIR)/dijkstra.c \
       $(ALGOS_DIR)/bellman_ford.c \
       $(ALGOS_DIR)/floyd_warshall.c \
       $(ALGOS_DIR)/bfs.c \
       $(ALGOS_DIR)/dfs.c \
       $(ALGOS_DIR)/kruskal.c \
       $(ALGOS_DIR)/prim.c \
       $(ALGOS_DIR)/knapsack_01.c \
       $(ALGOS_DIR)/knapsack_frac.c \
       $(ALGOS_DIR)/huffman.c \
       $(ALGOS_DIR)/kmp.c \
       $(ALGOS_DIR)/rabin_karp.c \
       $(ALGOS_DIR)/tsp_greedy.c \
       $(ALGOS_DIR)/tsp_brute.c \
       $(ALGOS_DIR)/job_seq.c \
       $(ALGOS_DIR)/activity_sel.c \
       $(ALGOS_DIR)/heap_sort.c \
       $(ALGOS_DIR)/lcs.c \
       $(ALGOS_DIR)/benchmark.c

# Linux ke liye shared library banao
all:
	$(CC) $(CFLAGS) $(JNI_INCLUDE) -shared -o libscae.so $(SRCS)
	@echo "✅ libscae.so compiled successfully"

# Windows ke liye DLL banao
windows:
	$(CC) $(CFLAGS) -I$(JAVA_HOME)/include -I$(JAVA_HOME)/include/win32 \
	      -shared -o scae.dll $(SRCS)
	@echo "✅ scae.dll compiled successfully"

# purani compiled files delete karo
clean:
	rm -f libscae.so scae.dll
	@echo "🧹 Cleaned compiled files"
```

## STEP I — Create empty data JSON files

Create these 10 files in data/ folder (empty arrays to start):

```json
// data/users.json
[]

// data/complaints.json
[]

// data/work_orders.json
[]

// data/emergencies.json
[]

// data/nodes.json
[]

// data/edges.json
[]

// data/notices.json
[]

// data/projects.json
[]

// data/services.json
[]

// data/sensor_logs.json
[]
```

## STEP J — Write .env.example

```env
PORT=8000
JWT_SECRET=scae-super-secret-key-minimum-32-characters-long
FRONTEND_URL=https://bucolic-biscotti-dfb4d5.netlify.app
```

## CHECKLIST — Before moving to Prompt 2:
- [ ] `javac ScaeServer.java handlers/HealthHandler.java utils/*.java middleware/*.java` → no errors
- [ ] `java ScaeServer` → "✅ SCAE Server started at http://localhost:8000" printed
- [ ] `GET http://localhost:8000/api/health` → `{ "success": true, "status": "ok" }`
- [ ] `ls data/` → 10 JSON files visible
- [ ] All 10 JSON files contain empty array `[]`
- [ ] `make` → libscae.so compiled (even if C files are empty stubs for now)

---

# ══════════════════════════════════════════════════════════
# PROMPT 2 — Auth Routes + Seed Data (users + city graph)
# ══════════════════════════════════════════════════════════
# Test after this → POST /api/auth/login → JWT token milega
# Data verify     → cat data/users.json → 3 users dikhne chahiye
#                   cat data/nodes.json → 13 nodes
#                   cat data/edges.json → 19 edges

[Paste MASTER CONTEXT above this line]

TASK: Write AuthHandler.java + seed all JSON data files (users, nodes, edges, all entities).

## STEP A — Seed data/users.json

```json
[
  {
    "id": "u001",
    "name": "City Commissioner",
    "email": "admin@scae.gov.in",
    "password": "$2a$12$LQv3c1yqBWVHxkd0LHAkCOYz6TiGqvENMKZp4N0m7oQrYbWuJ2l1e",
    "role": "admin",
    "zone": "City Hall",
    "created_at": "2025-01-01T00:00:00Z",
    "is_active": true
  },
  {
    "id": "u002",
    "name": "Aarav Sharma",
    "email": "citizen@scae.gov.in",
    "password": "$2a$12$LQv3c1yqBWVHxkd0LHAkCOYz6TiGqvENMKZp4N0m7oQrYbWuJ2l1e",
    "role": "citizen",
    "zone": "North Gate",
    "created_at": "2025-01-02T00:00:00Z",
    "is_active": true
  },
  {
    "id": "u003",
    "name": "Dept Officer Ravi",
    "email": "dept@scae.gov.in",
    "password": "$2a$12$LQv3c1yqBWVHxkd0LHAkCOYz6TiGqvENMKZp4N0m7oQrYbWuJ2l1e",
    "role": "dept",
    "zone": "Operations Center",
    "created_at": "2025-01-03T00:00:00Z",
    "is_active": true
  }
]
```
Note: All 3 passwords are bcrypt hash of "scae123" — login ke liye use karo.

## STEP B — Seed data/nodes.json (13 city nodes)

```json
[
  {"id": 0,  "label": "Central Junction", "x": 310, "y": 260, "zone": "Central"},
  {"id": 1,  "label": "North Gate",       "x": 310, "y": 80,  "zone": "North"},
  {"id": 2,  "label": "City Hall",        "x": 160, "y": 160, "zone": "West"},
  {"id": 3,  "label": "Market Square",    "x": 460, "y": 160, "zone": "East"},
  {"id": 4,  "label": "University",       "x": 100, "y": 260, "zone": "West"},
  {"id": 5,  "label": "Downtown",         "x": 460, "y": 260, "zone": "East"},
  {"id": 6,  "label": "Old Town",         "x": 160, "y": 360, "zone": "South-West"},
  {"id": 7,  "label": "River Road",       "x": 310, "y": 380, "zone": "South"},
  {"id": 8,  "label": "East Market",      "x": 460, "y": 360, "zone": "South-East"},
  {"id": 9,  "label": "Medical Hub",      "x": 220, "y": 180, "zone": "Central"},
  {"id": 10, "label": "Tech District",    "x": 400, "y": 180, "zone": "East"},
  {"id": 11, "label": "Industrial Zone",  "x": 100, "y": 380, "zone": "South-West"},
  {"id": 12, "label": "Sports Complex",   "x": 520, "y": 380, "zone": "South-East"}
]
```

## STEP C — Seed data/edges.json (19 city edges)

```json
[
  {"id": 0,  "node_u": 0, "node_v": 1,  "weight": 4, "is_closed": false},
  {"id": 1,  "node_u": 0, "node_v": 2,  "weight": 3, "is_closed": false},
  {"id": 2,  "node_u": 0, "node_v": 3,  "weight": 3, "is_closed": false},
  {"id": 3,  "node_u": 0, "node_v": 7,  "weight": 3, "is_closed": false},
  {"id": 4,  "node_u": 1, "node_v": 9,  "weight": 2, "is_closed": false},
  {"id": 5,  "node_u": 1, "node_v": 10, "weight": 2, "is_closed": false},
  {"id": 6,  "node_u": 2, "node_v": 4,  "weight": 2, "is_closed": false},
  {"id": 7,  "node_u": 2, "node_v": 9,  "weight": 2, "is_closed": false},
  {"id": 8,  "node_u": 3, "node_v": 5,  "weight": 2, "is_closed": false},
  {"id": 9,  "node_u": 3, "node_v": 10, "weight": 2, "is_closed": false},
  {"id": 10, "node_u": 4, "node_v": 6,  "weight": 2, "is_closed": false},
  {"id": 11, "node_u": 4, "node_v": 11, "weight": 3, "is_closed": false},
  {"id": 12, "node_u": 5, "node_v": 8,  "weight": 2, "is_closed": false},
  {"id": 13, "node_u": 6, "node_v": 7,  "weight": 2, "is_closed": false},
  {"id": 14, "node_u": 6, "node_v": 11, "weight": 2, "is_closed": false},
  {"id": 15, "node_u": 7, "node_v": 8,  "weight": 2, "is_closed": false},
  {"id": 16, "node_u": 7, "node_v": 12, "weight": 3, "is_closed": false},
  {"id": 17, "node_u": 8, "node_v": 12, "weight": 2, "is_closed": false},
  {"id": 18, "node_u": 9, "node_v": 10, "weight": 1, "is_closed": false}
]
```

## STEP D — Seed remaining data files

**data/notices.json:**
```json
[
  {"id": "n001", "title": "Road maintenance on MG Road — 2 Jan 2026", "date": "2026-01-02", "link": "#", "priority": 1, "created_by": "u001"},
  {"id": "n002", "title": "Water supply suspended North Zone — 3 Jan 2026", "date": "2026-01-03", "link": "#", "priority": 2, "created_by": "u001"},
  {"id": "n003", "title": "New complaint portal launched", "date": "2025-12-01", "link": "#", "priority": 0, "created_by": "u001"}
]
```

**data/projects.json:**
```json
[
  {"id": "p001", "name": "Road Repair Phase 1",     "cost": 120, "benefit": 85,  "category": "Infrastructure", "active": true},
  {"id": "p002", "name": "Solar Street Lights",      "cost": 80,  "benefit": 70,  "category": "Energy",         "active": true},
  {"id": "p003", "name": "Water Pipeline Upgrade",   "cost": 200, "benefit": 95,  "category": "Utilities",      "active": true},
  {"id": "p004", "name": "CCTV Network Expansion",  "cost": 150, "benefit": 80,  "category": "Safety",         "active": true},
  {"id": "p005", "name": "Public Park Development",  "cost": 60,  "benefit": 50,  "category": "Civic",          "active": true},
  {"id": "p006", "name": "Smart Traffic Signals",   "cost": 180, "benefit": 90,  "category": "Traffic",        "active": true}
]
```

**data/services.json:**
```json
[
  {"id": "s001", "name": "City General Hospital",    "zone": "Medical Hub",      "type": "Hospital",  "node_id": 9,  "phone": "0135-2710000"},
  {"id": "s002", "name": "Central Fire Station",     "zone": "Central",          "type": "Emergency", "node_id": 0,  "phone": "101"},
  {"id": "s003", "name": "North Gate Police Post",   "zone": "North",            "type": "Police",    "node_id": 1,  "phone": "100"},
  {"id": "s004", "name": "City Hall Admin Office",   "zone": "West",             "type": "Govt",      "node_id": 2,  "phone": "0135-2654321"},
  {"id": "s005", "name": "GEU University",           "zone": "West",             "type": "Education", "node_id": 4,  "phone": "0135-2770137"},
  {"id": "s006", "name": "Tech Park Clinic",         "zone": "East",             "type": "Hospital",  "node_id": 10, "phone": "0135-2711111"},
  {"id": "s007", "name": "River Road Bus Stand",     "zone": "South",            "type": "Transport", "node_id": 7,  "phone": "0135-2622222"},
  {"id": "s008", "name": "East Market Police Chowk", "zone": "South-East",      "type": "Police",    "node_id": 8,  "phone": "100"},
  {"id": "s009", "name": "Sports Complex Stadium",   "zone": "South-East",       "type": "Sports",    "node_id": 12, "phone": "0135-2733333"},
  {"id": "s010", "name": "Industrial Zone Clinic",   "zone": "South-West",       "type": "Hospital",  "node_id": 11, "phone": "0135-2744444"}
]
```

**data/complaints.json:**
```json
[
  {"id": "CMP-0001", "citizen_id": "u002", "category": "ROAD",   "description": "Pothole near North Gate signal", "urgency": 8, "zone": "North",   "status": "Pending",     "date_filed": "2026-04-20", "assigned_to": null, "notes": ""},
  {"id": "CMP-0002", "citizen_id": "u002", "category": "POWER",  "description": "Street light not working",       "urgency": 6, "zone": "Central", "status": "In Progress", "date_filed": "2026-04-21", "assigned_to": "u003", "notes": "Crew assigned"},
  {"id": "CMP-0003", "citizen_id": "u002", "category": "WATER",  "description": "Water pipe leakage on MG Road", "urgency": 9, "zone": "West",    "status": "Resolved",    "date_filed": "2026-04-22", "assigned_to": "u003", "notes": "Fixed on 23 Apr"}
]
```

**data/work_orders.json:**
```json
[
  {"id": "WO-001", "title": "Fix pothole North Gate", "zone": "North",   "urgency": 8, "deadline": "2026-04-28", "crew": "Crew Alpha", "status": "Pending",     "description": "Pothole repair near signal"},
  {"id": "WO-002", "title": "Street light repair",    "zone": "Central", "urgency": 6, "deadline": "2026-04-27", "crew": "Crew Beta",  "status": "In Progress", "description": "3 lights not working"},
  {"id": "WO-003", "title": "Water pipe repair MG",   "zone": "West",    "urgency": 9, "deadline": "2026-04-25", "crew": "Crew Gamma", "status": "Completed",   "description": "Leakage fixed"},
  {"id": "WO-004", "title": "CCTV install East Zone", "zone": "East",    "urgency": 5, "deadline": "2026-04-30", "crew": "Crew Delta", "status": "Pending",     "description": "4 new CCTV cameras"}
]
```

**data/emergencies.json:**
```json
[
  {"id": 1, "type": "Road Accident",  "zone": "North",   "urgency": 10, "time": "08:30", "crew": "Crew Alpha", "status": "Dispatched"},
  {"id": 2, "type": "Fire Outbreak",  "zone": "East",    "urgency": 9,  "time": "09:15", "crew": "Crew Beta",  "status": "Resolved"},
  {"id": 3, "type": "Power Failure",  "zone": "Central", "urgency": 7,  "time": "10:00", "crew": null,         "status": "Queued"},
  {"id": 4, "type": "Flood Alert",    "zone": "South",   "urgency": 8,  "time": "11:30", "crew": null,         "status": "Queued"},
  {"id": 5, "type": "Water Burst",    "zone": "West",    "urgency": 6,  "time": "12:00", "crew": "Crew Gamma", "status": "Dispatched"}
]
```

**data/sensor_logs.json:**
```json
[
  {"id": 1, "type": "traffic", "raw_data": "aabbbcccdddeeeffgg", "timestamp": "2026-04-25T08:00:00Z", "zone": "North"},
  {"id": 2, "type": "air",     "raw_data": "aaabbbccddeeee",     "timestamp": "2026-04-25T09:00:00Z", "zone": "Central"},
  {"id": 3, "type": "temp",    "raw_data": "aabbcddeeefff",      "timestamp": "2026-04-25T10:00:00Z", "zone": "East"}
]
```

## STEP E — Write handlers/AuthHandler.java

```java
// AuthHandler.java — login, profile routes handle karta hai
// POST /api/auth/login → JWT token deta hai
// GET  /api/auth/me    → current user ka profile deta hai

package handlers;

import com.sun.net.httpserver.HttpExchange;
import com.sun.net.httpserver.HttpHandler;
import utils.*;
import middleware.AuthMiddleware;
import java.io.*;

public class AuthHandler implements HttpHandler {

    public void handle(HttpExchange exchange) throws IOException {

        // CORS headers add karo
        CorsUtil.addCorsHeaders(exchange);
        if ("OPTIONS".equals(exchange.getRequestMethod())) {
            exchange.sendResponseHeaders(204, -1);
            return;
        }

        String path = exchange.getRequestURI().getPath();
        String method = exchange.getRequestMethod();

        // POST /api/auth/login → login karo
        if (path.equals("/api/auth/login") && method.equals("POST")) {
            handleLogin(exchange);

        // GET /api/auth/me → current user ka profile
        } else if (path.equals("/api/auth/me") && method.equals("GET")) {
            handleMe(exchange);

        } else {
            sendResponse(exchange, 404, "{\"success\":false,\"message\":\"Route not found\"}");
        }
    }

    private void handleLogin(HttpExchange exchange) throws IOException {

        // request body read karo
        InputStream is = exchange.getRequestBody();
        String body = new String(is.readAllBytes());

        // email aur password nikalo — simple string parsing
        String email = extractField(body, "email");
        String password = extractField(body, "password");

        if (email == null || password == null) {
            sendResponse(exchange, 400,
                "{\"success\":false,\"message\":\"Email aur password required hai\"}");
            return;
        }

        // users.json se user dhundo
        String usersJson = JsonUtil.readFile("users.json");

        // user find karo by email
        // Note: real implementation mein JSON parser use karo (org.json ya Gson)
        // yahan simple check ke liye string contains use karo
        if (!usersJson.contains("\"email\": \"" + email + "\"")) {
            sendResponse(exchange, 401,
                "{\"success\":false,\"message\":\"Invalid email ya password\"}");
            return;
        }

        // TODO: bcrypt password verify karo
        // abhi ke liye "scae123" hardcoded check karo
        if (!"scae123".equals(password)) {
            sendResponse(exchange, 401,
                "{\"success\":false,\"message\":\"Invalid email ya password\"}");
            return;
        }

        // role aur name nikalo user JSON se
        String role = extractUserField(usersJson, email, "role");
        String name = extractUserField(usersJson, email, "name");

        // JWT token generate karo
        String token = JwtUtil.generateToken(email, role, name);

        // response bhejo
        String response = "{"
            + "\"success\":true,"
            + "\"message\":\"Login successful\","
            + "\"data\":{"
            +   "\"token\":\"" + token + "\","
            +   "\"role\":\"" + role + "\","
            +   "\"name\":\"" + name + "\","
            +   "\"email\":\"" + email + "\""
            + "}}";

        sendResponse(exchange, 200, response);
    }

    private void handleMe(HttpExchange exchange) throws IOException {

        // token verify karo
        String payload = AuthMiddleware.verify(exchange);
        if (payload == null) {
            AuthMiddleware.sendUnauthorized(exchange);
            return;
        }

        // payload se user info nikalo
        String email = extractPayloadField(payload, "sub");
        String role  = extractPayloadField(payload, "role");
        String name  = extractPayloadField(payload, "name");

        String response = "{"
            + "\"success\":true,"
            + "\"message\":\"Profile fetched\","
            + "\"data\":{"
            +   "\"email\":\"" + email + "\","
            +   "\"role\":\"" + role + "\","
            +   "\"name\":\"" + name + "\""
            + "}}";

        sendResponse(exchange, 200, response);
    }

    // helper: JSON body se field value nikalo
    private String extractField(String json, String field) {
        String key = "\"" + field + "\"";
        int idx = json.indexOf(key);
        if (idx == -1) return null;
        int start = json.indexOf("\"", idx + key.length() + 1) + 1;
        int end   = json.indexOf("\"", start);
        if (start <= 0 || end <= 0) return null;
        return json.substring(start, end);
    }

    // helper: users array se specific user ka field nikalo
    private String extractUserField(String usersJson, String email, String field) {
        int emailIdx = usersJson.indexOf("\"" + email + "\"");
        if (emailIdx == -1) return "unknown";
        String userBlock = usersJson.substring(emailIdx);
        return extractField("{" + userBlock, field);
    }

    // helper: JWT payload se field nikalo
    private String extractPayloadField(String payload, String field) {
        String key = "\"" + field + "\":\"";
        int idx = payload.indexOf(key);
        if (idx == -1) return "unknown";
        int start = idx + key.length();
        int end = payload.indexOf("\"", start);
        return payload.substring(start, end);
    }

    // helper: response bhejo
    private void sendResponse(HttpExchange exchange, int code, String body) throws IOException {
        exchange.getResponseHeaders().set("Content-Type", "application/json");
        exchange.sendResponseHeaders(code, body.getBytes().length);
        OutputStream os = exchange.getResponseBody();
        os.write(body.getBytes());
        os.close();
    }
}
```

## CHECKLIST — Before moving to Prompt 3:
- [ ] `POST /api/auth/login` with `{ "email": "admin@scae.gov.in", "password": "scae123" }` → token milta hai
- [ ] `GET /api/auth/me` with Bearer token → name, role, email aata hai
- [ ] `cat data/users.json` → 3 users dikhte hai
- [ ] `cat data/nodes.json` → 13 nodes dikhte hai
- [ ] `cat data/edges.json` → 19 edges dikhte hai
- [ ] All 10 data files seeded properly

---

# ══════════════════════════════════════════════════════════
# PROMPT 3 — Graph Routes + C Algorithm Files (Graph Algorithms)
# ══════════════════════════════════════════════════════════
# Test after this → GET /api/graph/nodes → 13 nodes
#                   POST /api/algo/dijkstra → shortest path milegi
# C verify        → make → all graph C files compile hona chahiye

[Paste MASTER CONTEXT above this line]

TASK: Write GraphHandler.java + C files for all graph algorithms (Dijkstra, BFS, DFS,
Bellman-Ford, Floyd-Warshall, Kruskal, Prim) + JNI bridge setup.

## STEP A — Write algorithms/scae.h (shared struct definitions)

```c
// scae.h — SCAE ke saare shared structs yahan define hai
// ye header file saare C files mein include hogi

#ifndef SCAE_H
#define SCAE_H

#define MAX_V 50   // maximum vertices city graph mein
#define MAX_E 200  // maximum edges city graph mein
#define INF 99999  // infinity represent karne ke liye

// ek edge ko represent karta hai
typedef struct {
    int u;      // source node
    int v;      // destination node
    int weight; // road ka weight (km mein)
} Edge;

// algorithm ka result store karta hai
typedef struct {
    int path[MAX_V];      // shortest path nodes
    int path_len;         // path mein kitne nodes hai
    int distance;         // total distance
    int visited[MAX_V];   // BFS/DFS ke liye visited array
    int visited_len;      // kitne nodes visited hue
    int components[MAX_V][MAX_V]; // DFS components ke liye
    int comp_count;               // total components
    int mst_edges_u[MAX_V];       // MST edges source
    int mst_edges_v[MAX_V];       // MST edges destination
    int mst_edges_w[MAX_V];       // MST edges weight
    int mst_count;                // MST mein kitne edges hai
    int mst_total;                // MST ka total cost
    int matrix[MAX_V][MAX_V];     // Floyd-Warshall matrix ke liye
    long time_ns;                  // execution time nanoseconds mein
} AlgoResult;

#endif
```

## STEP B — Write algorithms/dijkstra.c

```c
// dijkstra.c — Dijkstra's Shortest Path Algorithm
// SCAE Navigation Engine — city map pe shortest path nikalta hai
// Time: O((V+E) log V) | Space: O(V)

#include "scae.h"
#include <limits.h>
#include <string.h>

// minimum distance wala unvisited node dhundo
static int minDist(int dist[], int visited[], int V) {
    int min = INF, idx = -1;
    // saare nodes check karo
    for (int v = 0; v < V; v++) {
        // sirf unvisited nodes mein se minimum dhundo
        if (!visited[v] && dist[v] <= min) {
            min = dist[v];
            idx = v;
        }
    }
    return idx;
}

// path traceback karo — destination se source tak
static int tracePath(int prev[], int src, int dst, AlgoResult *result) {
    // agar destination tak path nahi hai
    if (prev[dst] == -1 && dst != src) return 0;

    // path stack mein dalo
    int stack[MAX_V], top = 0;
    int curr = dst;
    while (curr != -1) {
        stack[top++] = curr;
        curr = prev[curr];
    }

    // stack ko ulta karke path banao
    result->path_len = 0;
    for (int i = top - 1; i >= 0; i--) {
        result->path[result->path_len++] = stack[i];
    }
    return 1;
}

// Dijkstra ka main function
// graph adjacency matrix se kaam karta hai
void dijkstra(int graph[MAX_V][MAX_V], int V, int src, int dst,
              int skip_edge_u, int skip_edge_v, AlgoResult *result) {

    int dist[MAX_V], visited[MAX_V], prev[MAX_V];

    // saare distances infinity se initialize karo
    for (int i = 0; i < V; i++) {
        dist[i] = INF;
        visited[i] = 0;
        prev[i] = -1;
    }

    // source node ka distance 0 hai
    dist[src] = 0;

    // V-1 iterations chalao
    for (int c = 0; c < V - 1; c++) {

        // minimum distance wala node lo
        int u = minDist(dist, visited, V);
        if (u == -1) break;

        // us node ko visited mark karo
        visited[u] = 1;

        // u ke saare neighbors check karo
        for (int v = 0; v < V; v++) {

            // skip closed edge — road closure simulation ke liye
            if ((u == skip_edge_u && v == skip_edge_v) ||
                (u == skip_edge_v && v == skip_edge_u)) continue;

            // edge hai aur v abhi visited nahi
            if (!visited[v] && graph[u][v] && dist[u] + graph[u][v] < dist[v]) {
                // shorter path mila — update karo
                dist[v] = dist[u] + graph[u][v];
                prev[v] = u;
            }
        }
    }

    // result fill karo
    result->distance = dist[dst];
    tracePath(prev, src, dst, result);
}
```

## STEP C — Write algorithms/bfs.c

```c
// bfs.c — Breadth First Search Algorithm
// SCAE Zone Reachability — N hops ke andar kaunse zones accessible hain
// Time: O(V+E) | Space: O(V)

#include "scae.h"
#include <string.h>

// BFS ka main function
// src se max_hops distance ke andar saare nodes dhundo
void bfs(int graph[MAX_V][MAX_V], int V, int src, int max_hops, AlgoResult *result) {

    int visited[MAX_V], hops[MAX_V];
    int queue[MAX_V], front = 0, rear = 0;

    // initialization — saab unvisited hai
    for (int i = 0; i < V; i++) {
        visited[i] = 0;
        hops[i] = -1;
    }

    // source node se start karo
    visited[src] = 1;
    hops[src] = 0;
    queue[rear++] = src;

    result->visited_len = 0;
    result->path[result->visited_len++] = src; // source BFS order mein pehle aata hai

    // queue empty hone tak chalao
    while (front < rear) {
        int u = queue[front++]; // queue se pehla element nikalo

        // agar max hops ho gaye toh aage mat jao
        if (hops[u] >= max_hops) continue;

        // u ke saare neighbors check karo
        for (int v = 0; v < V; v++) {
            // edge hai aur v visited nahi
            if (graph[u][v] && !visited[v]) {
                visited[v] = 1;
                hops[v] = hops[u] + 1;
                queue[rear++] = v; // queue mein dalo
                result->path[result->visited_len++] = v; // result mein save karo
            }
        }
    }

    // total reachable count
    result->distance = result->visited_len;
}
```

## STEP D — Write algorithms/dfs.c

```c
// dfs.c — Depth First Search Algorithm
// SCAE Disaster Simulation — disconnected zones detect karta hai
// Time: O(V+E) | Space: O(V)

#include "scae.h"
#include <string.h>

static int global_visited[MAX_V];
static int current_comp[MAX_V];
static int current_comp_size;

// recursive DFS helper
static void dfsHelper(int graph[MAX_V][MAX_V], int V, int u) {

    // u ko visited mark karo
    global_visited[u] = 1;

    // current component mein add karo
    current_comp[current_comp_size++] = u;

    // u ke saare neighbors recursively visit karo
    for (int v = 0; v < V; v++) {
        if (graph[u][v] && !global_visited[v]) {
            dfsHelper(graph, V, v);
        }
    }
}

// DFS se connected components dhundo
// disabled_nodes[] = wo nodes jo disaster mein offline ho gaye
void dfs_components(int graph[MAX_V][MAX_V], int V,
                    int disabled_nodes[], int disabled_count,
                    AlgoResult *result) {

    // initialization
    for (int i = 0; i < V; i++) global_visited[i] = 0;

    // disabled nodes ko already visited mark karo
    for (int i = 0; i < disabled_count; i++) {
        if (disabled_nodes[i] < V) global_visited[disabled_nodes[i]] = 1;
    }

    result->comp_count = 0;

    // saare unvisited nodes se DFS start karo
    for (int u = 0; u < V; u++) {
        if (!global_visited[u]) {
            // naya component shuru hua
            current_comp_size = 0;
            dfsHelper(graph, V, u);

            // component result mein save karo
            for (int i = 0; i < current_comp_size; i++) {
                result->components[result->comp_count][i] = current_comp[i];
            }
            result->visited[result->comp_count] = current_comp_size; // component size
            result->comp_count++;
        }
    }
}
```

## STEP E — Write algorithms/floyd_warshall.c

```c
// floyd_warshall.c — Floyd-Warshall All-Pairs Shortest Path
// SCAE Admin Panel — city ke saare node pairs ke beech shortest distance
// Time: O(V^3) | Space: O(V^2)

#include "scae.h"
#include <string.h>

// Floyd-Warshall ka main function
void floyd_warshall(int graph[MAX_V][MAX_V], int V, AlgoResult *result) {

    // distance matrix copy karo graph se
    for (int i = 0; i < V; i++) {
        for (int j = 0; j < V; j++) {
            if (i == j) {
                result->matrix[i][j] = 0;       // apne aap se distance 0
            } else if (graph[i][j] != 0) {
                result->matrix[i][j] = graph[i][j]; // direct edge ka weight
            } else {
                result->matrix[i][j] = INF;     // koi direct edge nahi
            }
        }
    }

    // teen nested loops — ye Floyd-Warshall ka core hai
    for (int k = 0; k < V; k++) {           // intermediate node k
        for (int i = 0; i < V; i++) {       // source i
            for (int j = 0; j < V; j++) {   // destination j
                // kya i→k→j path i→j se shorter hai?
                if (result->matrix[i][k] != INF &&
                    result->matrix[k][j] != INF &&
                    result->matrix[i][k] + result->matrix[k][j] < result->matrix[i][j]) {
                    // haan — shorter path mila
                    result->matrix[i][j] = result->matrix[i][k] + result->matrix[k][j];
                }
            }
        }
    }

    result->distance = 0; // success indicator
}
```

## STEP F — Write algorithms/kruskal.c

```c
// kruskal.c — Kruskal's Minimum Spanning Tree Algorithm
// SCAE Power Grid Optimizer — minimum cable se saare zones connect karo
// Time: O(E log E) | Space: O(V)

#include "scae.h"
#include <stdlib.h>
#include <string.h>

// DSU (Disjoint Set Union) — Union-Find structure
static int parent[MAX_V], rnk[MAX_V];

// DSU find — path compression ke saath
static int find(int x) {
    // path compression — parent seedha root se connect karo
    if (parent[x] != x) parent[x] = find(parent[x]);
    return parent[x];
}

// DSU union — union by rank ke saath
static void unite(int a, int b) {
    a = find(a); b = find(b);
    if (a == b) return; // already ek hi component mein hain

    // chhoti rank wale ko bade ke neeche rakho
    if (rnk[a] < rnk[b]) { int t = a; a = b; b = t; }
    parent[b] = a;
    if (rnk[a] == rnk[b]) rnk[a]++;
}

// edges sort karne ke liye comparator
static int cmpEdge(const void *a, const void *b) {
    return ((Edge*)a)->weight - ((Edge*)b)->weight;
}

// Kruskal ka main function
void kruskal(Edge edges[], int E, int V, AlgoResult *result) {

    // DSU initialize karo — har node apna parent hai
    for (int i = 0; i < V; i++) {
        parent[i] = i;
        rnk[i] = 0;
    }

    // edges ko weight ke order mein sort karo (ascending)
    qsort(edges, E, sizeof(Edge), cmpEdge);

    result->mst_count = 0;
    result->mst_total = 0;

    // sorted edges mein se MST edges chuno
    for (int i = 0; i < E; i++) {
        int u = edges[i].u, v = edges[i].v, w = edges[i].weight;

        // agar same component mein nahi hai — cycle nahi banega
        if (find(u) != find(v)) {
            unite(u, v);         // dono ko merge karo
            // MST edge save karo
            result->mst_edges_u[result->mst_count] = u;
            result->mst_edges_v[result->mst_count] = v;
            result->mst_edges_w[result->mst_count] = w;
            result->mst_total += w;
            result->mst_count++;

            // V-1 edges ke baad MST complete hoti hai
            if (result->mst_count == V - 1) break;
        }
    }
}
```

## STEP G — Write algorithms/prim.c

```c
// prim.c — Prim's Minimum Spanning Tree Algorithm
// SCAE Power Grid Optimizer — Kruskal ka alternative, same result
// Time: O((V+E) log V) | Space: O(V)

#include "scae.h"
#include <string.h>

// Prim ka main function
// graph adjacency matrix se kaam karta hai
void prim(int graph[MAX_V][MAX_V], int V, AlgoResult *result) {

    int key[MAX_V];     // har vertex ke liye minimum edge weight
    int inMST[MAX_V];   // vertex MST mein hai ya nahi
    int parent[MAX_V];  // MST tree structure ke liye

    // initialization
    for (int i = 0; i < V; i++) {
        key[i] = INF;
        inMST[i] = 0;
        parent[i] = -1;
    }

    // source node se shuru karo — vertex 0
    key[0] = 0;

    result->mst_count = 0;
    result->mst_total = 0;

    // V-1 edges tak chalao
    for (int c = 0; c < V - 1; c++) {

        // minimum key wala unvisited vertex dhundo
        int u = -1, minKey = INF;
        for (int v = 0; v < V; v++) {
            if (!inMST[v] && key[v] < minKey) {
                minKey = key[v];
                u = v;
            }
        }

        if (u == -1) break; // koi vertex nahi mila

        // u ko MST mein add karo
        inMST[u] = 1;

        // u ka parent edge MST mein save karo
        if (parent[u] != -1) {
            result->mst_edges_u[result->mst_count] = parent[u];
            result->mst_edges_v[result->mst_count] = u;
            result->mst_edges_w[result->mst_count] = graph[parent[u]][u];
            result->mst_total += graph[parent[u]][u];
            result->mst_count++;
        }

        // u ke neighbors ke keys update karo
        for (int v = 0; v < V; v++) {
            // agar edge hai aur v MST mein nahi aur ye edge chhoti hai
            if (graph[u][v] && !inMST[v] && graph[u][v] < key[v]) {
                key[v] = graph[u][v]; // key update karo
                parent[v] = u;         // parent set karo
            }
        }
    }
}
```

## STEP H — Write algorithms/bellman_ford.c

```c
// bellman_ford.c — Bellman-Ford Shortest Path Algorithm
// SCAE Road Closure Simulation — road band hone pe alternate route dhundo
// Time: O(VE) | Space: O(V)

#include "scae.h"
#include <string.h>

// Bellman-Ford ka main function
// edges[] array se kaam karta hai — ek specific edge skip kar sakta hai
void bellman_ford(Edge edges[], int E, int V, int src, int dst,
                  int skip_u, int skip_v, AlgoResult *result) {

    int dist[MAX_V], prev[MAX_V];

    // initialization — saab infinity se
    for (int i = 0; i < V; i++) {
        dist[i] = INF;
        prev[i] = -1;
    }

    // source ka distance 0 hai
    dist[src] = 0;

    // V-1 relaxation passes karo
    for (int pass = 0; pass < V - 1; pass++) {
        // saare edges ke liye relaxation
        for (int i = 0; i < E; i++) {
            int u = edges[i].u, v = edges[i].v, w = edges[i].weight;

            // ye edge skip karo — road closure simulate karne ke liye
            if ((u == skip_u && v == skip_v) || (u == skip_v && v == skip_u)) continue;

            // forward relaxation
            if (dist[u] != INF && dist[u] + w < dist[v]) {
                dist[v] = dist[u] + w;
                prev[v] = u;
            }
            // backward relaxation — undirected graph hai
            if (dist[v] != INF && dist[v] + w < dist[u]) {
                dist[u] = dist[v] + w;
                prev[u] = v;
            }
        }
    }

    // result fill karo
    result->distance = dist[dst];
    result->path_len = 0;

    // path traceback karo
    if (dist[dst] != INF) {
        int stack[MAX_V], top = 0;
        int curr = dst;
        while (curr != -1) {
            stack[top++] = curr;
            curr = prev[curr];
        }
        for (int i = top - 1; i >= 0; i--) {
            result->path[result->path_len++] = stack[i];
        }
    }
}
```

## STEP I — Write handlers/GraphHandler.java (stubs for now — full in Prompt 6)

Write GraphHandler.java with these endpoints:
```
GET  /api/graph/nodes              → nodes.json read karke return karo
GET  /api/graph/edges              → edges.json read karke return karo
PUT  /api/graph/edges/{id}/close   → edge ka is_closed = true karo (Admin only)
PUT  /api/graph/edges/{id}/open    → edge ka is_closed = false karo (Admin only)
POST /api/graph/nodes              → naya node add karo (Admin only)
POST /api/graph/edges              → naya edge add karo (Admin only)
```

## CHECKLIST — Before moving to Prompt 4:
- [ ] `GET /api/graph/nodes` → 13 nodes array milti hai
- [ ] `GET /api/graph/edges` → 19 edges array milti hai
- [ ] `PUT /api/graph/edges/3/close` with admin token → edge 3 is_closed=true ho jata hai
- [ ] `cat algorithms/dijkstra.c` → complete C file hai
- [ ] `make` → compiles successfully (stub JNI file ke saath)
- [ ] All 7 C algorithm files hai (dijkstra, bfs, dfs, bellman_ford, floyd_warshall, kruskal, prim)

---

# ══════════════════════════════════════════════════════════
# PROMPT 4 — JNI Bridge + AlgoHandler + Algorithm Endpoints (Graph)
# ══════════════════════════════════════════════════════════
# Test after this →
#   POST /api/algo/dijkstra      → path array milega
#   POST /api/algo/bfs           → reachable nodes milenge
#   POST /api/algo/dfs/components → connected components milenge
#   GET  /api/algo/floyd-warshall → distance matrix milega
#   GET  /api/algo/mst/kruskal   → MST edges milenge
#   GET  /api/algo/mst/prim      → MST edges milenge

[Paste MASTER CONTEXT above this line]

TASK: Write ScaeJNI.c (JNI bridge) + Java native method declarations + AlgoHandler.java
for all graph algorithm endpoints.

## STEP A — Write algorithms/ScaeJNI.c (JNI wrapper)

```c
// ScaeJNI.c — JNI bridge — Java se C functions call hoti hain yahan
// ye file Java ke native methods ko C implementations se connect karti hai
// IMPORTANT: ye file compile hone ke liye JAVA_HOME set hona chahiye

#include <jni.h>
#include "scae.h"
#include <string.h>
#include <stdlib.h>

// saare algorithm headers include karo
// (implementations already .c files mein hain — linker connect karega)

// =============================================
// DIJKSTRA JNI wrapper
// Java se: algo.runDijkstra(graph, V, src, dst, skipU, skipV)
// =============================================
JNIEXPORT jintArray JNICALL Java_handlers_AlgoHandler_runDijkstra(
    JNIEnv *env, jclass cls,
    jintArray jGraph, jint V, jint src, jint dst,
    jint skipU, jint skipV) {

    // Java int array ko C array mein convert karo
    jint *graphArr = (*env)->GetIntArrayElements(env, jGraph, NULL);

    // graph 2D matrix banao
    int graph[MAX_V][MAX_V];
    for (int i = 0; i < V; i++)
        for (int j = 0; j < V; j++)
            graph[i][j] = graphArr[i * V + j];

    (*env)->ReleaseIntArrayElements(env, jGraph, graphArr, 0);

    // Dijkstra algorithm chalao
    AlgoResult result;
    memset(&result, 0, sizeof(result));
    dijkstra(graph, V, src, dst, skipU, skipV, &result);

    // result Java array mein convert karo
    // format: [distance, path_len, node1, node2, node3, ...]
    int returnSize = 2 + result.path_len;
    jintArray jResult = (*env)->NewIntArray(env, returnSize);
    jint *resultArr = malloc(returnSize * sizeof(jint));
    resultArr[0] = result.distance;
    resultArr[1] = result.path_len;
    for (int i = 0; i < result.path_len; i++)
        resultArr[2 + i] = result.path[i];
    (*env)->SetIntArrayRegion(env, jResult, 0, returnSize, resultArr);
    free(resultArr);

    return jResult;
}

// =============================================
// BFS JNI wrapper
// Java se: algo.runBFS(graph, V, src, maxHops)
// =============================================
JNIEXPORT jintArray JNICALL Java_handlers_AlgoHandler_runBFS(
    JNIEnv *env, jclass cls,
    jintArray jGraph, jint V, jint src, jint maxHops) {

    jint *graphArr = (*env)->GetIntArrayElements(env, jGraph, NULL);
    int graph[MAX_V][MAX_V];
    for (int i = 0; i < V; i++)
        for (int j = 0; j < V; j++)
            graph[i][j] = graphArr[i * V + j];
    (*env)->ReleaseIntArrayElements(env, jGraph, graphArr, 0);

    AlgoResult result;
    memset(&result, 0, sizeof(result));
    bfs(graph, V, src, maxHops, &result);

    // reachable nodes return karo
    jintArray jResult = (*env)->NewIntArray(env, result.visited_len + 1);
    jint *arr = malloc((result.visited_len + 1) * sizeof(jint));
    arr[0] = result.visited_len;
    for (int i = 0; i < result.visited_len; i++) arr[i + 1] = result.path[i];
    (*env)->SetIntArrayRegion(env, jResult, 0, result.visited_len + 1, arr);
    free(arr);
    return jResult;
}

// =============================================
// FLOYD-WARSHALL JNI wrapper
// =============================================
JNIEXPORT jintArray JNICALL Java_handlers_AlgoHandler_runFloydWarshall(
    JNIEnv *env, jclass cls,
    jintArray jGraph, jint V) {

    jint *graphArr = (*env)->GetIntArrayElements(env, jGraph, NULL);
    int graph[MAX_V][MAX_V];
    for (int i = 0; i < V; i++)
        for (int j = 0; j < V; j++)
            graph[i][j] = graphArr[i * V + j];
    (*env)->ReleaseIntArrayElements(env, jGraph, graphArr, 0);

    AlgoResult result;
    memset(&result, 0, sizeof(result));
    floyd_warshall(graph, V, &result);

    // V*V matrix return karo flat array mein
    jintArray jResult = (*env)->NewIntArray(env, V * V);
    jint *arr = malloc(V * V * sizeof(jint));
    for (int i = 0; i < V; i++)
        for (int j = 0; j < V; j++)
            arr[i * V + j] = result.matrix[i][j];
    (*env)->SetIntArrayRegion(env, jResult, 0, V * V, arr);
    free(arr);
    return jResult;
}

// =============================================
// KRUSKAL JNI wrapper
// =============================================
JNIEXPORT jintArray JNICALL Java_handlers_AlgoHandler_runKruskal(
    JNIEnv *env, jclass cls,
    jintArray jEdgesU, jintArray jEdgesV, jintArray jEdgesW,
    jint E, jint V) {

    jint *eu = (*env)->GetIntArrayElements(env, jEdgesU, NULL);
    jint *ev = (*env)->GetIntArrayElements(env, jEdgesV, NULL);
    jint *ew = (*env)->GetIntArrayElements(env, jEdgesW, NULL);

    Edge edges[MAX_E];
    for (int i = 0; i < E; i++) {
        edges[i].u = eu[i];
        edges[i].v = ev[i];
        edges[i].weight = ew[i];
    }
    (*env)->ReleaseIntArrayElements(env, jEdgesU, eu, 0);
    (*env)->ReleaseIntArrayElements(env, jEdgesV, ev, 0);
    (*env)->ReleaseIntArrayElements(env, jEdgesW, ew, 0);

    AlgoResult result;
    memset(&result, 0, sizeof(result));
    kruskal(edges, E, V, &result);

    // MST result return karo: [total_cost, count, u1, v1, w1, u2, v2, w2, ...]
    int returnSize = 2 + result.mst_count * 3;
    jintArray jResult = (*env)->NewIntArray(env, returnSize);
    jint *arr = malloc(returnSize * sizeof(jint));
    arr[0] = result.mst_total;
    arr[1] = result.mst_count;
    for (int i = 0; i < result.mst_count; i++) {
        arr[2 + i * 3]     = result.mst_edges_u[i];
        arr[2 + i * 3 + 1] = result.mst_edges_v[i];
        arr[2 + i * 3 + 2] = result.mst_edges_w[i];
    }
    (*env)->SetIntArrayRegion(env, jResult, 0, returnSize, arr);
    free(arr);
    return jResult;
}
```

## STEP B — Write handlers/AlgoHandler.java (Graph section)

```java
// AlgoHandler.java — algorithm endpoints handle karta hai
// ye handler C algorithms ko JNI ke zariye call karta hai

package handlers;

import com.sun.net.httpserver.HttpExchange;
import com.sun.net.httpserver.HttpHandler;
import utils.*;
import middleware.AuthMiddleware;
import java.io.*;

public class AlgoHandler implements HttpHandler {

    // C shared library load karo — JNI ke liye zaroori hai
    static {
        try {
            System.loadLibrary("scae"); // libscae.so ya scae.dll
            System.out.println("✅ C Algorithm Library loaded via JNI");
        } catch (UnsatisfiedLinkError e) {
            System.err.println("❌ JNI Library load nahi hua: " + e.getMessage());
        }
    }

    // ye native methods C mein implement hain — JNI se connect hoti hain
    private static native int[] runDijkstra(int[] graph, int V, int src, int dst, int skipU, int skipV);
    private static native int[] runBFS(int[] graph, int V, int src, int maxHops);
    private static native int[] runBellmanFord(int[] edgesU, int[] edgesV, int[] edgesW, int E, int V, int src, int dst, int skipU, int skipV);
    private static native int[] runFloydWarshall(int[] graph, int V);
    private static native int[] runKruskal(int[] edgesU, int[] edgesV, int[] edgesW, int E, int V);
    private static native int[] runPrim(int[] graph, int V);
    private static native int[] runTSPGreedy(int[] graph, int V, int[] checkpoints, int cpCount);
    private static native int[] runDijkstraBFS(int[] graph, int V, int src, int maxHops);
    private static native int runKnapsack01(int[] costs, int[] benefits, int n, int W);
    private static native int[] runHuffman(String input);
    private static native int[] runKMP(String text, String pattern);
    private static native int[] runJobScheduler(int[] deadlines, int[] profits, int n);
    private static native double[] runBenchmark(int n);

    public void handle(HttpExchange exchange) throws IOException {
        CorsUtil.addCorsHeaders(exchange);
        if ("OPTIONS".equals(exchange.getRequestMethod())) {
            exchange.sendResponseHeaders(204, -1);
            return;
        }

        String path = exchange.getRequestURI().getPath();
        String method = exchange.getRequestMethod();

        // ========================
        // DIJKSTRA — POST /api/algo/dijkstra
        // ========================
        if (path.equals("/api/algo/dijkstra") && method.equals("POST")) {
            handleDijkstra(exchange);

        // ========================
        // BFS — POST /api/algo/bfs
        // ========================
        } else if (path.equals("/api/algo/bfs") && method.equals("POST")) {
            handleBFS(exchange);

        // ========================
        // BELLMAN-FORD — POST /api/algo/bellman-ford
        // ========================
        } else if (path.equals("/api/algo/bellman-ford") && method.equals("POST")) {
            handleBellmanFord(exchange);

        // ========================
        // FLOYD-WARSHALL — GET /api/algo/floyd-warshall (Admin only)
        // ========================
        } else if (path.equals("/api/algo/floyd-warshall") && method.equals("GET")) {
            String payload = AuthMiddleware.verify(exchange);
            if (!AuthMiddleware.isAdmin(payload)) { AuthMiddleware.sendForbidden(exchange); return; }
            handleFloydWarshall(exchange);

        // ========================
        // MST KRUSKAL — GET /api/algo/mst/kruskal (Admin only)
        // ========================
        } else if (path.equals("/api/algo/mst/kruskal") && method.equals("GET")) {
            String payload = AuthMiddleware.verify(exchange);
            if (!AuthMiddleware.isAdmin(payload)) { AuthMiddleware.sendForbidden(exchange); return; }
            handleMST(exchange, "kruskal");

        // ========================
        // MST PRIM — GET /api/algo/mst/prim (Admin only)
        // ========================
        } else if (path.equals("/api/algo/mst/prim") && method.equals("GET")) {
            String payload = AuthMiddleware.verify(exchange);
            if (!AuthMiddleware.isAdmin(payload)) { AuthMiddleware.sendForbidden(exchange); return; }
            handleMST(exchange, "prim");

        // ========================
        // DFS COMPONENTS — POST /api/algo/dfs/components (Admin)
        // ========================
        } else if (path.equals("/api/algo/dfs/components") && method.equals("POST")) {
            String payload = AuthMiddleware.verify(exchange);
            if (!AuthMiddleware.isAdmin(payload)) { AuthMiddleware.sendForbidden(exchange); return; }
            handleDFSComponents(exchange);

        } else {
            // remaining algo endpoints Prompt 5 mein aayenge
            sendResponse(exchange, 404, "{\"success\":false,\"message\":\"Algorithm endpoint not found\"}");
        }
    }

    private void handleDijkstra(HttpExchange exchange) throws IOException {
        // request body read karo
        String body = new String(exchange.getRequestBody().readAllBytes());
        int src = extractInt(body, "source");
        int dst = extractInt(body, "destination");
        int skipU = extractIntOr(body, "skip_edge_u", -1);
        int skipV = extractIntOr(body, "skip_edge_v", -1);

        // edges.json se graph build karo
        int[] graphMatrix = buildGraphMatrix(13); // 13 nodes hain

        // C function call karo JNI se
        int[] result = runDijkstra(graphMatrix, 13, src, dst, skipU, skipV);

        // result format karo
        int distance = result[0];
        int pathLen = result[1];
        StringBuilder pathArr = new StringBuilder("[");
        for (int i = 0; i < pathLen; i++) {
            if (i > 0) pathArr.append(",");
            pathArr.append(result[2 + i]);
        }
        pathArr.append("]");

        String response = "{"
            + "\"success\":true,"
            + "\"message\":\"Dijkstra completed\","
            + "\"data\":{"
            +   "\"path\":" + pathArr + ","
            +   "\"distance\":" + distance + ","
            +   "\"algorithm\":\"Dijkstra\","
            +   "\"complexity\":\"O((V+E) log V)\""
            + "}}";

        sendResponse(exchange, 200, response);
    }

    // TODO: handleBFS, handleBellmanFord, handleFloydWarshall, handleMST, handleDFSComponents
    // same pattern — body read karo, C call karo, response format karo

    // helper: edges.json se adjacency matrix banao
    private int[] buildGraphMatrix(int V) throws IOException {
        String edgesJson = JsonUtil.readFile("edges.json");
        int[] matrix = new int[V * V];
        // edge parsing — simple string approach
        // TODO: proper JSON parser use karo (org.json ya Gson add karo)
        return matrix;
    }

    // helper: int field extract karo JSON body se
    private int extractInt(String json, String field) {
        String key = "\"" + field + "\":";
        int idx = json.indexOf(key);
        if (idx == -1) return 0;
        int start = idx + key.length();
        while (start < json.length() && json.charAt(start) == ' ') start++;
        int end = start;
        while (end < json.length() && (Character.isDigit(json.charAt(end)) || json.charAt(end) == '-')) end++;
        try { return Integer.parseInt(json.substring(start, end)); }
        catch (Exception e) { return 0; }
    }

    private int extractIntOr(String json, String field, int defaultVal) {
        try { return extractInt(json, field); } catch (Exception e) { return defaultVal; }
    }

    private void sendResponse(HttpExchange exchange, int code, String body) throws IOException {
        exchange.getResponseHeaders().set("Content-Type", "application/json");
        exchange.sendResponseHeaders(code, body.getBytes().length);
        OutputStream os = exchange.getResponseBody();
        os.write(body.getBytes());
        os.close();
    }
}
```

## CHECKLIST — Before moving to Prompt 5:
- [ ] `make` → ScaeJNI.c compiles with all graph C files
- [ ] `POST /api/algo/dijkstra` with `{ "source": 0, "destination": 9 }` → path milta hai
- [ ] `POST /api/algo/bfs` with `{ "source": 0, "max_hops": 2 }` → reachable nodes milte hai
- [ ] `GET /api/algo/floyd-warshall` with admin token → 13x13 matrix milta hai
- [ ] `GET /api/algo/mst/kruskal` with admin token → MST edges milte hai
- [ ] JNI library load message console pe aata hai

---

# ══════════════════════════════════════════════════════════
# PROMPT 5 — DP + Greedy + String Matching Algorithms (C + Java)
# ══════════════════════════════════════════════════════════
# Test after this →
#   POST /api/algo/knapsack       → selected projects milenge
#   POST /api/algo/huffman        → compression result milega
#   POST /api/algo/kmp            → pattern match positions milenge
#   POST /api/algo/job-scheduler  → gantt chart milega
#   GET  /api/algo/benchmark      → sorting algorithm times milenge

[Paste MASTER CONTEXT above this line]

TASK: Write C files for DP + Greedy + String algorithms +
extend AlgoHandler.java with these endpoints.

## STEP A — Write algorithms/knapsack_01.c

```c
// knapsack_01.c — 0/1 Knapsack Dynamic Programming
// SCAE Budget Allocator — best infrastructure projects select karo budget mein
// Time: O(nW) | Space: O(nW)

#include "scae.h"
#include <string.h>

// 0/1 Knapsack ka main function
// cost[] = projects ka cost, benefit[] = projects ka benefit
// n = projects ki count, W = total budget
// returns optimal benefit aur selected[] mein selected projects
int knapsack_01(int cost[], int benefit[], int n, int W, int selected[]) {

    // DP table — dp[i][w] = i items mein se W budget pe max benefit
    int dp[51][1001];  // max 50 projects, max 1000 budget
    memset(dp, 0, sizeof(dp));

    // DP table fill karo bottom-up
    for (int i = 1; i <= n; i++) {
        for (int w = 0; w <= W; w++) {
            // project i include nahi karo
            dp[i][w] = dp[i-1][w];

            // project i include karo agar cost fit ho raha hai
            if (cost[i-1] <= w) {
                int withItem = dp[i-1][w - cost[i-1]] + benefit[i-1];
                if (withItem > dp[i][w]) {
                    // include karna better hai
                    dp[i][w] = withItem;
                }
            }
        }
    }

    // traceback karo — kaunse projects select hue
    int w = W;
    for (int i = n; i > 0; i--) {
        if (dp[i][w] != dp[i-1][w]) {
            // ye project select hua hai
            selected[i-1] = 1;
            w -= cost[i-1]; // budget kam karo
        } else {
            selected[i-1] = 0;
        }
    }

    // maximum benefit return karo
    return dp[n][W];
}
```

## STEP B — Write algorithms/huffman.c

```c
// huffman.c — Huffman Coding Greedy Algorithm
// SCAE Sensor Logger — sensor data compress karta hai
// Time: O(n log n) | Space: O(n)

#include "scae.h"
#include <stdio.h>
#include <string.h>
#include <stdlib.h>

#define MAX_CHAR 256

// Huffman tree node
typedef struct HuffNode {
    char ch;           // character
    int freq;          // frequency
    int left, right;   // child indices (-1 if leaf)
} HuffNode;

// Huffman result structure
typedef struct {
    char codes[MAX_CHAR][MAX_CHAR]; // har character ka code
    int original_bits;              // original size bits mein
    int compressed_bits;            // compressed size bits mein
    int frequencies[MAX_CHAR];      // character frequencies
} HuffResult;

// Huffman main function — string compress karo
void huffman(const char *input, HuffResult *result) {

    int n = strlen(input);
    result->original_bits = n * 8; // original = 8 bits per character

    // character frequencies count karo
    memset(result->frequencies, 0, sizeof(result->frequencies));
    for (int i = 0; i < n; i++) {
        result->frequencies[(unsigned char)input[i]]++;
    }

    // min-heap banao unique characters se
    HuffNode nodes[MAX_CHAR * 2];
    int nodeCount = 0;
    int heap[MAX_CHAR * 2], heapSize = 0;

    for (int i = 0; i < MAX_CHAR; i++) {
        if (result->frequencies[i] > 0) {
            nodes[nodeCount].ch = (char)i;
            nodes[nodeCount].freq = result->frequencies[i];
            nodes[nodeCount].left = nodes[nodeCount].right = -1;
            heap[heapSize++] = nodeCount++;
        }
    }

    // simple bubble sort for heap (small input ke liye theek hai)
    // production mein real min-heap use karo
    for (int i = 0; i < heapSize; i++)
        for (int j = 0; j < heapSize - i - 1; j++)
            if (nodes[heap[j]].freq > nodes[heap[j+1]].freq) {
                int t = heap[j]; heap[j] = heap[j+1]; heap[j+1] = t;
            }

    // Huffman tree build karo
    while (heapSize > 1) {
        // do smallest frequency nodes lo
        int l = heap[0], r = heap[1];
        heapSize -= 2;
        for (int i = 0; i < heapSize; i++) heap[i] = heap[i+2];

        // parent node banao
        nodes[nodeCount].ch = '\0';
        nodes[nodeCount].freq = nodes[l].freq + nodes[r].freq;
        nodes[nodeCount].left = l;
        nodes[nodeCount].right = r;

        // parent ko heap mein insert karo (sorted position pe)
        int pos = heapSize;
        heap[heapSize++] = nodeCount;
        while (pos > 0 && nodes[heap[pos]].freq < nodes[heap[pos-1]].freq) {
            int t = heap[pos]; heap[pos] = heap[pos-1]; heap[pos-1] = t;
            pos--;
        }
        nodeCount++;
    }

    // codes generate karo — tree traverse karo
    // TODO: recursive DFS for code generation
    // Simplified: har character ke liye code "" set karo
    result->compressed_bits = n; // placeholder — real implementation mein calculate karo
}
```

## STEP C — Write algorithms/kmp.c

```c
// kmp.c — Knuth-Morris-Pratt String Matching Algorithm
// SCAE Complaint Search — complaint text mein fast keyword search
// Time: O(n+m) | Space: O(m)

#include "scae.h"
#include <string.h>
#include <stdio.h>

// LPS (Longest Proper Prefix Suffix) array compute karo
// ye KMP ka preprocessing step hai — O(m) time
void computeLPS(const char *pattern, int m, int lps[]) {

    // pehle character ka LPS hamesha 0 hota hai
    lps[0] = 0;
    int len = 0;  // pichla longest prefix-suffix ki length
    int i = 1;

    while (i < m) {
        if (pattern[i] == pattern[len]) {
            // match mila — length badhao
            lps[i++] = ++len;
        } else if (len != 0) {
            // match nahi — pichle LPS pe jao
            len = lps[len - 1];
            // i ko increment mat karo
        } else {
            // len == 0 aur match nahi
            lps[i++] = 0;
        }
    }
}

// KMP search — text mein pattern ke saare occurrences dhundo
// positions[] mein match positions store hoti hain
// returns total match count
int kmp_search(const char *text, int n, const char *pattern, int m,
               int positions[], int *comparisons) {

    int lps[MAX_V * 10];  // pattern ke liye LPS array
    computeLPS(pattern, m, lps);

    int matchCount = 0;
    *comparisons = 0;
    int i = 0, j = 0;  // i = text pointer, j = pattern pointer

    while (i < n) {
        (*comparisons)++;  // comparison count karo

        if (text[i] == pattern[j]) {
            // characters match — dono aage jao
            i++; j++;
        }

        if (j == m) {
            // pura pattern match ho gaya
            positions[matchCount++] = i - j;  // match position save karo
            j = lps[j - 1];  // next match ke liye reset karo
        } else if (i < n && text[i] != pattern[j]) {
            // mismatch — LPS use karke backtrack karo
            if (j != 0) {
                j = lps[j - 1];
            } else {
                i++;  // j == 0 pe sirf i aage jao
            }
        }
    }

    return matchCount;
}
```

## STEP D — Write algorithms/job_seq.c

```c
// job_seq.c — Job Sequencing with Deadlines (Greedy)
// SCAE Emergency Dispatch — deadline ke andar maximum profit jobs schedule karo
// Time: O(n^2) | Space: O(n)

#include "scae.h"
#include <stdlib.h>
#include <string.h>

// jobs sort karne ke liye — profit ke order mein (descending)
typedef struct {
    int id;
    int deadline;
    int profit;
} Job;

static int cmpJob(const void *a, const void *b) {
    return ((Job*)b)->profit - ((Job*)a)->profit; // descending profit order
}

// Job Sequencing main function
// returns scheduled job IDs aur total profit
int job_sequencing(int deadlines[], int profits[], int n,
                   int scheduled[], int *total_profit) {

    Job jobs[MAX_V];
    for (int i = 0; i < n; i++) {
        jobs[i].id = i;
        jobs[i].deadline = deadlines[i];
        jobs[i].profit = profits[i];
    }

    // jobs ko profit ke descending order mein sort karo
    qsort(jobs, n, sizeof(Job), cmpJob);

    // max deadline dhundo
    int maxDeadline = 0;
    for (int i = 0; i < n; i++)
        if (jobs[i].deadline > maxDeadline) maxDeadline = jobs[i].deadline;

    // time slots banao — -1 matlab khaali slot
    int slots[MAX_V];
    for (int i = 0; i < maxDeadline; i++) slots[i] = -1;

    *total_profit = 0;
    int scheduledCount = 0;

    // har job ke liye latest available slot dhundo
    for (int i = 0; i < n; i++) {
        int d = jobs[i].deadline - 1; // 0-indexed

        // deadline se pehle khaali slot dhundo
        for (int j = d; j >= 0; j--) {
            if (slots[j] == -1) {
                // slot available hai — job schedule karo
                slots[j] = jobs[i].id;
                scheduled[scheduledCount++] = jobs[i].id;
                *total_profit += jobs[i].profit;
                break;
            }
        }
    }

    return scheduledCount;
}
```

## STEP E — Extend handlers/AlgoHandler.java with DP/Greedy endpoints

Add these methods to existing AlgoHandler.java:

```java
// Knapsack endpoint — POST /api/algo/knapsack
private void handleKnapsack(HttpExchange exchange) throws IOException {
    // request body read karo
    String body = new String(exchange.getRequestBody().readAllBytes());
    int budget = extractInt(body, "budget");

    // projects.json se data read karo
    String projectsJson = JsonUtil.readFile("projects.json");

    // C function call karo
    // int optimalBenefit = runKnapsack01(costs, benefits, n, budget);

    // response format karo aur bhejo
    String response = "{\"success\":true,\"message\":\"Knapsack completed\",\"data\":{\"budget\":" + budget + "}}";
    sendResponse(exchange, 200, response);
}

// Huffman endpoint — POST /api/algo/huffman
private void handleHuffman(HttpExchange exchange) throws IOException {
    // sensor_logs.json se data lo ya body se input lo
    // C huffman function call karo
    // compression ratio return karo
}

// KMP endpoint — POST /api/algo/kmp
private void handleKMP(HttpExchange exchange) throws IOException {
    // request body se text aur pattern lo
    // C kmp_search function call karo
    // match positions return karo
}

// Job Scheduler — POST /api/algo/job-scheduler
private void handleJobScheduler(HttpExchange exchange) throws IOException {
    // work_orders.json se jobs lo
    // C job_sequencing function call karo
    // Gantt chart format mein return karo
}

// Benchmark — GET /api/algo/benchmark?n=1000
private void handleBenchmark(HttpExchange exchange) throws IOException {
    // C benchmark.c se saare 7 sorting algorithms run karo
    // time results return karo
}
```

## STEP F — Write algorithms/lcs.c, algorithms/rabin_karp.c, algorithms/tsp_greedy.c

Write each with Hinglish comments, same pattern as above.

**lcs.c:** LCS DP — O(mn) — complaint patterns compare karo
**rabin_karp.c:** Rolling hash — O(n+m) avg — pattern detection
**tsp_greedy.c:** Nearest neighbour — O(n^2) — patrol route
**activity_sel.c:** Greedy intervals — O(n log n) — maintenance scheduling
**heap_sort.c:** Max-heap sort — O(n log n) — emergency triage
**benchmark.c:** All 7 sorting algorithms timed with clock() — benchmark console ke liye

## CHECKLIST — Before moving to Prompt 6:
- [ ] `POST /api/algo/knapsack` with budget + project_ids → selected projects milte hai
- [ ] `POST /api/algo/huffman` with input string → codes aur compression ratio milte hai
- [ ] `POST /api/algo/kmp` with text + pattern → match positions milte hai
- [ ] `POST /api/algo/job-scheduler` with jobs array → gantt chart milta hai
- [ ] `GET /api/algo/benchmark?n=1000` → 7 algorithms ke times milte hai
- [ ] All 13 C algorithm files complete hain

---

# ══════════════════════════════════════════════════════════
# PROMPT 6 — CRUD Routes (Complaints + Work Orders + Emergencies)
# ══════════════════════════════════════════════════════════
# Test after this →
#   GET  /api/complaints           → 3 complaints milenge
#   POST /api/complaints           → naya complaint file hoga
#   GET  /api/work-orders/sorted   → urgency sorted list milegi
#   POST /api/emergencies/dispatch-next → top urgency emergency dispatch hogi

[Paste MASTER CONTEXT above this line]

TASK: Write ComplaintHandler.java + WorkOrderHandler.java + EmergencyHandler.java
with full CRUD + JSON file read/write.

## STEP A — Write handlers/ComplaintHandler.java

```
Handle these routes:
GET  /api/complaints                → complaints.json read karo
                                      Admin: saab complaints milte hai
                                      Citizen: sirf apne complaints (JWT se email match)
POST /api/complaints                → naya complaint add karo
                                      id = "CMP-" + (count+1) padded to 4 digits
                                      status default = "Pending"
GET  /api/complaints/{id}           → single complaint by ID
PUT  /api/complaints/{id}/status    → status update karo (Admin/Dept only)
DELETE /api/complaints/{id}         → delete karo (Admin only)
GET  /api/complaints/sorted?by=urgency → heap-sort karke return karo
GET  /api/complaints/search?q=road  → KMP string search use karo description mein
```

Pattern for all CRUD operations:
```java
// GET: file read karo → JSON return karo
// POST: file read karo → new item add karo → file write karo → response bhejo
// PUT: file read karo → item dhundo → update karo → file write karo
// DELETE: file read karo → item remove karo → file write karo
```

## STEP B — Write handlers/WorkOrderHandler.java

```
GET  /api/work-orders               → work_orders.json read karo (Admin/Dept)
POST /api/work-orders               → naya work order create karo
GET  /api/work-orders/{id}          → single work order
PUT  /api/work-orders/{id}          → update
PUT  /api/work-orders/{id}/complete → status = "Completed" (Dept only)
DELETE /api/work-orders/{id}        → delete (Admin only)
GET  /api/work-orders/sorted?by=urgency → heap_sort.c se sort karke return karo
POST /api/work-orders/assign-top    → top urgency job assign karo crew ko
```

## STEP C — Write handlers/EmergencyHandler.java

```
GET  /api/emergencies                    → emergencies.json read karo
POST /api/emergencies                    → naya emergency create karo
POST /api/emergencies/generate           → 10 random emergencies generate karo
PUT  /api/emergencies/dispatch-next      → status=Queued mein se max urgency wala dispatch karo
                                           heap_sort.c use karo priority ke liye
PUT  /api/emergencies/{id}/resolve       → status = "Resolved" karo
GET  /api/emergencies/sorted?by=urgency  → urgency se heap sort karke return karo
```

## STEP D — Response formats

**Complaint create response:**
```json
{
  "success": true,
  "message": "Complaint filed. Reference: CMP-0004",
  "data": {
    "id": "CMP-0004",
    "status": "Pending",
    "date_filed": "2026-04-25"
  }
}
```

**Work order sorted response:**
```json
{
  "success": true,
  "message": "Work orders sorted by urgency",
  "data": [
    { "id": "WO-003", "title": "Water pipe repair", "urgency": 9, "status": "Completed" },
    { "id": "WO-001", "title": "Fix pothole", "urgency": 8, "status": "Pending" }
  ]
}
```

## CHECKLIST — Before moving to Prompt 7:
- [ ] `GET /api/complaints` with admin token → 3 complaints milte hai
- [ ] `POST /api/complaints` → naya CMP-0004 create hota hai, file mein save hota hai
- [ ] `PUT /api/complaints/CMP-0001/status` → status update hoti hai
- [ ] `GET /api/complaints/sorted?by=urgency` → urgency 9 pehle aata hai
- [ ] `GET /api/complaints/search?q=pothole` → CMP-0001 milta hai
- [ ] `PUT /api/emergencies/dispatch-next` → top priority emergency dispatched hoti hai
- [ ] `GET /api/work-orders/sorted?by=urgency` → correctly sorted list milti hai

---

# ══════════════════════════════════════════════════════════
# PROMPT 7 — Remaining Routes + Deployment + Frontend Connection
# ══════════════════════════════════════════════════════════
# Test after this →
#   GET /api/citizens/search?q=u001  → citizen profile milega
#   GET /api/stats/summary           → dashboard counters milenge
#   GET /api/notices                 → public notices milenge
#   Render deploy → production URL pe health check pass hoga

[Paste MASTER CONTEXT above this line]

TASK: Write remaining handlers + render.yaml + frontend connection steps + full test.

## STEP A — Write handlers/CitizenHandler.java (Admin only)

```
GET /api/citizens              → users.json se role=citizen filter karo
GET /api/citizens/search?q=    → name ya email se search karo (hash-based — O(1) average)
GET /api/citizens/{id}         → single citizen profile + unke complaints
```

## STEP B — Write handlers/ProjectHandler.java

```
GET    /api/projects           → projects.json return karo (Admin only)
POST   /api/projects           → naya project add karo
PUT    /api/projects/{id}      → project update karo
DELETE /api/projects/{id}      → active = false karo (soft delete)
```

## STEP C — Write handlers/NoticeHandler.java

```
GET    /api/notices            → notices.json return karo (Public — no auth)
POST   /api/notices            → naya notice add karo (Admin only)
PUT    /api/notices/{id}       → notice update karo (Admin only)
DELETE /api/notices/{id}       → notice delete karo (Admin only)
```

## STEP D — Write handlers/ServiceHandler.java

```
GET /api/services              → services.json return karo (Public)
GET /api/services/search?q=hospital → binary search use karo name pe
GET /api/services/zone/{zone}  → zone se filter karo
POST /api/services             → naya service add karo (Admin only)
DELETE /api/services/{id}      → service delete karo (Admin only)
```

## STEP E — Write handlers/StatsHandler.java

```
GET /api/stats/summary        → Public — homepage ke counters
  Return:
  {
    "complaints_resolved": count of resolved complaints,
    "zones_covered": 13 (nodes count),
    "active_work_orders": count of non-completed work orders,
    "citizens_registered": count of citizen users
  }

GET /api/stats/complaints     → Admin — complaints breakdown by zone + category
GET /api/stats/work-orders    → Admin — work orders stats
```

## STEP F — Write render.yaml

```yaml
# render.yaml — Render.com deployment config
# ye file Render ko batata hai ki backend kaise build aur run karna hai

services:
  - type: web
    name: scae-backend
    env: java
    plan: free

    # build command — C files compile karo aur Java compile karo
    buildCommand: |
      apt-get install -y gcc make
      make
      javac -cp ".:lib/*" ScaeServer.java handlers/*.java utils/*.java middleware/*.java

    # server start karo
    startCommand: java -Djava.library.path=. ScaeServer

    # environment variables — Render dashboard mein set karo
    envVars:
      - key: PORT
        value: 8000
      - key: FRONTEND_URL
        value: https://bucolic-biscotti-dfb4d5.netlify.app
      - key: JWT_SECRET
        generateValue: true
```

## STEP G — Run full test script

```bash
# bash script — scae_test_all.sh
# sab endpoints test karo ek ek karke

BASE="http://localhost:8000"
PASS=0
FAIL=0

echo "🏙️  SCAE Backend — Full API Test"
echo "══════════════════════════════════════════════"

# 1. Health check
STATUS=$(curl -s -o /dev/null -w "%{http_code}" $BASE/api/health)
if [ "$STATUS" = "200" ]; then echo "✅ GET /api/health"; PASS=$((PASS+1))
else echo "❌ GET /api/health → $STATUS"; FAIL=$((FAIL+1)); fi

# 2. Login
TOKEN=$(curl -s -X POST $BASE/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email":"admin@scae.gov.in","password":"scae123"}' | grep -o '"token":"[^"]*"' | cut -d'"' -f4)

if [ -n "$TOKEN" ]; then echo "✅ POST /api/auth/login → token received"; PASS=$((PASS+1))
else echo "❌ POST /api/auth/login → no token"; FAIL=$((FAIL+1)); fi

# 3. Graph nodes
STATUS=$(curl -s -o /dev/null -w "%{http_code}" $BASE/api/graph/nodes)
if [ "$STATUS" = "200" ]; then echo "✅ GET /api/graph/nodes"; PASS=$((PASS+1))
else echo "❌ GET /api/graph/nodes"; FAIL=$((FAIL+1)); fi

# 4. Dijkstra
STATUS=$(curl -s -o /dev/null -w "%{http_code}" -X POST $BASE/api/algo/dijkstra \
  -H "Content-Type: application/json" \
  -d '{"source":0,"destination":9}')
if [ "$STATUS" = "200" ]; then echo "✅ POST /api/algo/dijkstra"; PASS=$((PASS+1))
else echo "❌ POST /api/algo/dijkstra"; FAIL=$((FAIL+1)); fi

# ... (remaining 51 tests follow same pattern)

echo "══════════════════════════════════════════════"
echo "Total tested  : $((PASS+FAIL)) endpoints"
echo "✅ PASS       : $PASS"
echo "❌ FAIL       : $FAIL"
```

## FINAL CHECKLIST — Backend Complete ✅
- [ ] `bash scae_test_all.sh` → 50+ tests pass
- [ ] `GET /api/stats/summary` → correct counts milte hai
- [ ] `GET /api/services/search?q=hospital` → binary search se hospitals milte hai
- [ ] `GET /api/notices` → 3 notices milte hai (no auth required)
- [ ] `make` → libscae.so compiles with all 13 algorithm C files
- [ ] render.yaml present hai project root mein
- [ ] All 10 data JSON files seeded with sample data

---

# ══════════════════════════════════════════════════════════
# DEPLOYMENT — After all 7 prompts complete
# ══════════════════════════════════════════════════════════

```bash
# 1. GitHub pe push karo
git add .
git commit -m "SCAE Backend complete — 55 endpoints, 13 C algorithms, JNI bridge"
git push origin main

# 2. Render.com pe deploy karo
#    render.com → New Web Service → Connect GitHub → scae-backend select karo
#    Build command: check karo render.yaml se auto-pick hoga
#    Start command: check karo render.yaml se auto-pick hoga

# 3. Render mein ye env vars set karo (Dashboard → Environment)
#    FRONTEND_URL = https://bucolic-biscotti-dfb4d5.netlify.app
#    JWT_SECRET   = (koi bhi 32+ character secret)
#    PORT         = 8000

# 4. Deploy hone ke baad Render URL copy karo (example):
#    https://scae-backend.onrender.com

# 5. Test karo production URL pe
#    GET https://scae-backend.onrender.com/api/health → success true aana chahiye
```

---

# ══════════════════════════════════════════════════════════
# FRONTEND ↔ BACKEND CONNECTION — Step by Step Guide
# ══════════════════════════════════════════════════════════

## How to connect your friend's frontend to your backend:

### STEP 1 — api.js file update karo (your friend ka kaam)

Friend ke project mein ek file hogi `js/api.js` ya similar.
Us file mein BASE_URL update karo:

```javascript
// api.js — frontend ka centralized API caller
// ye file saare backend calls handle karti hai

const BASE_URL = "https://scae-backend.onrender.com/api"; // apna Render URL daalo

// JWT token sessionStorage mein store hota hai
function getToken() {
    return sessionStorage.getItem("scae_token");
}

// generic fetch helper — har API call yahan se hoti hai
async function apiCall(endpoint, method = "GET", body = null) {
    const options = {
        method,
        headers: {
            "Content-Type": "application/json",
            // agar token hai toh Authorization header add karo
            ...(getToken() && { "Authorization": "Bearer " + getToken() })
        }
    };
    if (body) options.body = JSON.stringify(body);

    const response = await fetch(BASE_URL + endpoint, options);
    return response.json();
}

// ========================
// AUTH CALLS
// ========================
async function login(email, password) {
    const result = await apiCall("/auth/login", "POST", { email, password });
    if (result.success) {
        // token store karo
        sessionStorage.setItem("scae_token", result.data.token);
        sessionStorage.setItem("scae_role", result.data.role);
        sessionStorage.setItem("scae_name", result.data.name);
    }
    return result;
}

// ========================
// GRAPH CALLS
// ========================
async function getNodes() {
    return apiCall("/graph/nodes");
}

async function getEdges() {
    return apiCall("/graph/edges");
}

// ========================
// ALGORITHM CALLS
// ========================
async function runDijkstra(source, destination) {
    return apiCall("/algo/dijkstra", "POST", { source, destination });
}

async function runMST(type = "kruskal") {
    return apiCall("/algo/mst/" + type);
}

async function runKnapsack(budget, project_ids) {
    return apiCall("/algo/knapsack", "POST", { budget, project_ids });
}

// ========================
// COMPLAINT CALLS
// ========================
async function getComplaints() {
    return apiCall("/complaints");
}

async function fileComplaint(data) {
    return apiCall("/complaints", "POST", data);
}

// ========================
// STATS CALLS
// ========================
async function getStats() {
    return apiCall("/stats/summary");
}
```

### STEP 2 — index.html ka login form connect karo

```javascript
// index.html mein login button ka onClick handler
document.getElementById("loginBtn").addEventListener("click", async () => {
    const email = document.getElementById("emailInput").value;
    const password = document.getElementById("passwordInput").value;

    const result = await login(email, password);

    if (result.success) {
        // role ke hisaab se page redirect karo
        if (result.data.role === "admin") {
            window.location.href = "admin.html";
        } else if (result.data.role === "citizen") {
            window.location.href = "citizen.html";
        } else {
            window.location.href = "dept.html";
        }
    } else {
        alert("Login failed: " + result.message);
    }
});
```

### STEP 3 — admin.html city map connect karo

```javascript
// admin.js mein map load hone ke waqt graph data fetch karo
async function initMap() {
    // graph data backend se lo
    const nodesResult = await getNodes();
    const edgesResult = await getEdges();

    if (nodesResult.success && edgesResult.success) {
        // MapEngine ko real data do — hardcoded data replace ho jayega
        MapEngine.init(nodesResult.data.nodes, edgesResult.data.edges);
    }
}

// Dijkstra button click handler
document.getElementById("findRouteBtn").addEventListener("click", async () => {
    const src = parseInt(document.getElementById("sourceSelect").value);
    const dst = parseInt(document.getElementById("destSelect").value);

    const result = await runDijkstra(src, dst);

    if (result.success) {
        // path animate karo map pe
        MapEngine.animatePath(result.data.path, result.data.distance);
    }
});
```

### STEP 4 — Hardcoded data replace karo

Friend ke JS files mein hardcoded arrays milenge like:
```javascript
// BEFORE (hardcoded data — ye replace karna hai)
const complaintsData = [
    { id: "CMP-001", ... },
    { id: "CMP-002", ... }
];
```

Replace karo with:
```javascript
// AFTER (backend se real data)
let complaintsData = [];
async function loadComplaints() {
    const result = await getComplaints();
    if (result.success) {
        complaintsData = result.data;
        renderComplaintsTable(complaintsData); // table render karo
    }
}
loadComplaints(); // page load pe call karo
```

### STEP 5 — Final testing order

```
1. Backend Render pe deploy karo
2. GET https://scae-backend.onrender.com/api/health → ok aana chahiye
3. Friend ke project mein BASE_URL update karo
4. index.html pe login karo: admin@scae.gov.in / scae123
5. Admin dashboard → city map load hona chahiye
6. "Find Route" click karo → Dijkstra result aana chahiye
7. Citizen login karo: citizen@scae.gov.in / scae123
8. Complaint file karo → database mein save hona chahiye
9. Dept login karo: dept@scae.gov.in / scae123
10. Work orders sorted list → urgency order mein hona chahiye
```

### STEP 6 — Troubleshooting common issues

```
ISSUE 1: CORS Error in browser
FIX: Render env mein FRONTEND_URL = exact Netlify URL (trailing slash nahi hona chahiye)

ISSUE 2: Token not working
FIX: sessionStorage.getItem("scae_token") check karo — null nahi hona chahiye

ISSUE 3: JNI Library load nahi hua
FIX: render.yaml mein buildCommand mein `make` pehle run hona chahiye

ISSUE 4: JSON file not found
FIX: data/ folder Render pe exist karna chahiye — .gitignore mein add mat karo

ISSUE 5: Algorithm returns wrong result
FIX: C file mein edge cases check karo — INF values correctly handle ho rahi hain
```

---
*Last updated: April 2025 | SCAE — Node Navigators | Smart City Algorithm Engine*
