# Dependency remediation and host trust review

Date: 2026-10-28 (Asia/Seoul). Base: fdfb7a1. Status: dependency patch prepared; repository trust review is preliminary, not a malware clearance.

## Dependency results

| Scope | Before | After |
|---|---|---|
| Web | 20 vulnerable package entries | 0 (npm audit after remediation) |
| API recon runtime | 10 vulnerable package entries | 0 (npm audit after remediation) |
| Go module advisories | 11 unique | 1 module-only; 0 reachable |

Counts represent dependency entries or unique Go advisories, not independent exploitable attack paths. Web lockfile updates include Next.js, sharp, Zod, HTTP/MCP and build dependencies. puppeteer-core moves from 23.11.1 to 25.16.0; browser compatibility remains untested. shadcn generator CLI was removed; its MIT CSS was preserved locally with its license.

Go: compress 1.18.7, x/crypto 0.58.0, x/net 0.62.0, related transitive updates; minimum Go 1.26.10. Remaining [GO-2026-5932](https://pkg.go.dev/vuln/GO-2026-5932) concerns unmaintained OpenPGP packages. Scanner finds the module, but no affected imported package or reachable symbol. This is not a finding-free module inventory.

## Host trust findings (not proof of malicious intent)

1. **Upstream executable replacement**: selfupdate/github.go sets Repo to Autumn-27/artex. selfupdate/stage.go downloads a binary and checksum from that same publisher and executes smokeTest before staging. Matching SHA256 proves consistency, not independent publisher trust. June fork updates can be replaced by upstream code.
2. **Third-party container distribution**: docker-compose.yml defaults to ghcr.io/jiwoochris/artex-global:latest. A local source patch does not patch or authenticate that image. install.sh/update.sh pull and run it.
3. **Host execution authority**: agent/mainagent.go and agent/chat.go configure permission.ModeBypass (guard hooks exist separately). server/customtool.go execPython passes os.Environ() to Python. norma v0.4.3 tool/bash.go, tool/task.go and tool/shell_session.go execute subprocesses under the host account; working directories do not provide OS isolation. This permits exposure of accessible files/environment if malicious instructions or code execute.
4. **Additional downloads**: norma tool/ripgrep.go can npm-install a global binary; Dockerfile pipes NodeSource setup into bash and installs unpinned latest Playwright packages. install.sh can pipe get.docker.com into sh. These extend trust beyond the checked commit.

No conclusive deliberate credential theft/backdoor was established in the inspected paths. This was targeted source triage, not exhaustive history, dependency-source, binary, persistence, or network-forensic analysis. Do not infer that no malicious code exists.

## Validation and exposure limits

Completed before host-trust concern was raised: frontend 13 tests passed; static export with type checking passed; i18n and rendered HTML language checks passed; biome check passed with existing warnings; Go tests and vet passed (DB integration paths can skip without PostgreSQL). npm installs used --ignore-scripts. Build/test code nevertheless ran on this host before trust triage; this was not isolated execution. No application server, installer, upstream release binary, Docker image, or company target scan was launched by this session. No further repository code execution was performed after the concern was raised.

Not verified: browser/Puppeteer runtime, live employee usage, deployed instance, container OS/global dependency vulnerabilities, complete dependency maliciousness, and host compromise status. Existing Dependabot alerts API returned 403: alerts disabled; account scope also insufficient. No account settings changed.

A new dependency-security workflow checks both npm roots and reachable Go vulnerabilities on pushes, pull requests and weekly. It is prepared but remote execution is not yet verified.

## Immediate recommendation

Keep this patch in draft review. Before operational use, audit and disable/replace upstream executable and image trust paths, and run only inside an isolated disposable environment without personal files, cloud credentials, Docker socket, or production network access. Further verification must occur there. No merge or deployment has been performed.
