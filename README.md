I pasted this into you myself, wire THIS coding agent up to my Experiential
Labs gateway, so my model calls route through https://api.experientiallabs.ai and show up in my usage.

Ground rules: never print my full key (first 8 characters at most), ask me
before you edit any config file or shell profile, and if you cannot do a step
(no file access, settings live in a GUI), print the exact manual steps for me
instead. Print what you're doing at each step.

1. Securely ask me for my API key.
   Use EXPLABS_API_KEY for my gateway key. I can create one at https://platform.experientiallabs.ai/api-keys.
   Treat the key as a secret: never commit it, store it outside the config I approve,
   or echo it back (first 8 characters at most).

2. Prove the key works before touching any config:
   GET https://api.experientiallabs.ai/v1/models with header "Authorization: Bearer <key>" -> 200 and
   the model slugs I can call. Remember the list; I'll pick models from it.

3. Apply the Claude Code integration:
   You cannot repoint yourself mid-session, and switching auth under a running
   session causes mixed-state errors. First prove the lane without touching any
   config, via one isolated subprocess run from this project (so my MCP servers
   and plugins load and their tool schemas ride the request. Tool schemas are
   where provider strictness bites, so a bare hello proves too little):
     ANTHROPIC_BASE_URL="https://api.experientiallabs.ai" ANTHROPIC_API_KEY="$EXPLABS_API_KEY" \
       ANTHROPIC_MODEL="<slug I pick>" \
       claude -p "list your available tools, then read ./README.md and reply
       with its first heading"
   Pass only if that run completes with tools listed and a real tool use.
   Then offer me four placements and apply the one I pick:
   (a) append to my shell profile, or (b) print for one-off use:
     export ANTHROPIC_BASE_URL="https://api.experientiallabs.ai"        # no /v1 suffix
     export ANTHROPIC_API_KEY="$EXPLABS_API_KEY"
     export ANTHROPIC_MODEL="<slug I pick>"
   For (a), securely load EXPLABS_API_KEY in every fresh shell before that reference
   from my local secret store or a protected, gitignored env file. Never put the
   literal key in my shell profile or printed instructions.
   or (c) a named profile beside my normal Claude Code (ask me for the name):
   its own config dir ~/.claude-<name> with these three in settings.json "env"
   plus "model": "<slug I pick>". The config writer must read EXPLABS_API_KEY
   internally and write its resolved value as ANTHROPIC_API_KEY: settings.json
   does not expand shell references. Never write the literal "$EXPLABS_API_KEY"
   as that JSON value or put the actual value in tool inputs. Add a shell alias
     alias claude-<name>='CLAUDE_CONFIG_DIR=~/.claude-<name> claude'
   A fresh config dir starts with the first-run onboarding, so skip it: copy
   .claude.json from an existing profile dir that uses a gateway key (theme,
   trusted projects), else write {"hasCompletedOnboarding": true}; then set
   customApiKeyResponses to {"approved": ["<last 20 chars of my key>"],
   "rejected": []} so the custom-API-key prompt never appears. Never copy
   .credentials.json or a claude.ai sign-in into it.
   (d) keep my claude.ai sign-in (for claude.ai features such as Artifacts,
   which refuse while ANTHROPIC_API_KEY is set) and still route every model call
   through the gateway: in the config dir I name, put in settings.json "env"
   ANTHROPIC_BASE_URL and the model variables as above, NO ANTHROPIC_API_KEY,
   and the key as a header instead:
     set ANTHROPIC_CUSTOM_HEADERS to "x-api-key: " plus the key read inside the
     config writer from EXPLABS_API_KEY; never include its value in tool inputs.
   (Claude Code 2.1.227 or later; check claude --version first). The gateway
   reads x-api-key ahead of the Authorization header on /v1/messages, so the
   request is billed to my key and the sign-in token is dropped at the gateway.
   After the change I run /login once in a new session and pick the Claude
   account. Prove it with the same subprocess run, this time with
   ANTHROPIC_CUSTOM_HEADERS in place of ANTHROPIC_API_KEY.
   For (a), (b) and (c) use ANTHROPIC_API_KEY, never ANTHROPIC_AUTH_TOKEN: an
   existing claude.ai sign-in can outrank AUTH_TOKEN and send the gateway an
   OAuth token instead.
   Then tell me to start a NEW session from a shell with those set (for (c),
   a new terminal or after re-sourcing my shell profile). Name the
   model by its gateway slug exactly as GET https://api.experientiallabs.ai/v1/models lists it (the DOT
   form, e.g. claude-fable-5.1; Anthropic's dashed ids like claude-fable-5-1
   also work and are recorded under the dot form, but dated ids answer 403
   model_not_granted). If the slug is not an Anthropic model, also
   export CLAUDE_CODE_MAX_CONTEXT_TOKENS=<max_input_tokens from
   GET https://api.experientiallabs.ai/api/models/<slug>, rounded down to the nearest 50k>. Claude Code
   assumes 200k for names it does not know and compacts early. NEVER use
   context_window for that pin: on most models (every maker but Anthropic and
   Gemini) it is a total window the provider counts the model's maximum output
   inside, so a session pinned to it never compacts before the provider refuses
   it and /compact then fails the same way. max_input_tokens already accounts for this per maker, so it is
   always the right pin. Warn me plainly: if a session ever shows
   "Not logged in · Please run /login", do NOT run /login (it starts Anthropic
   OAuth that overrides the gateway). Restart with --resume, and if it
   persists, my <config dir>/.claude.json remembers a "No" to the custom-API-key
   prompt under customApiKeyResponses.rejected: answer Yes or move the entry to
   approved. Images work on image-capable routes and are rejected by name on
   text-only ones; extended thinking passes through on all-Anthropic routes
   and is translated to reasoning effort elsewhere.

4. Verify end to end. Make one tiny completion with the surface you configured
   ("reply with the single word: ok", small max output), via your own next
   model call if you now route through the gateway, otherwise via curl with my
   key. Then tell me it landed and that I can watch every call at
   https://platform.experientiallabs.ai/observability/requests .

5. Recap exactly what you changed (files and values, key shown as xpl_ prefix
   only) so I can undo it later.
