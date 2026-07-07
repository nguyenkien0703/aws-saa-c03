# DevOps VietNam — Crawl Report

**Target site:** https://devops.vn/  
**Crawl date:** 2026-07-04  
**Output directory:** `./devopsVN-crawl/`  

---

## 1. Summary

- **Pages crawled successfully:** 3865  (files `0001.md` … `3865.md`)
- **Sitemap URLs declared:** 1044
- **Sitemap URLs crawled:** 1043 / 1044
- **Failed / unreachable URLs:** 38 (all confirmed broken on the source site — see §4)
- **Discovery method:** `sitemap_index.xml` (all 10 sub-sitemaps) + recursive BFS following every internal link to closure

## 2. Coverage by section

| Section | Pages |
|---|---|
| /tag/ | 989 |
| /archived-events/ | 948 |
| /posts/ | 564 |
| /new/ | 392 |
| /2025/ | 280 |
| /event/ | 92 |
| /2026/ | 82 |
| /2024/ | 70 |
| /user/ | 65 |
| /author/ | 64 |
| /venue/ | 39 |
| /organizer/ | 34 |
| /2023/ | 29 |
| /2022/ | 26 |
| /2021/ | 24 |
| /2017/ | 21 |
| /2018/ | 17 |
| /2019/ | 14 |
| /2020/ | 14 |
| /en/ | 13 |
| /series/ | 12 |
| /vi/ | 11 |
| /series_group/ | 8 |
| /resources/ | 8 |
| /expert-insights/ | 4 |
| /webinar/ | 4 |
| /linux-foundation-cashback-fourth26/ | 2 |
| /linux-foundation-cashback-hbdk826/ | 2 |
| /linux-foundation-cashback-jprime26/ | 2 |
| /linux-foundation-cashback-mm26/ | 2 |
| /linux-foundation-cashback-pi-day/ | 2 |
| /linux-foundation-cashback/ | 2 |
| /home/ | 1 |
| /news/ | 1 |
| /newsletter/ | 1 |
| /about/ | 1 |
| /add-post/ | 1 |
| /cheat-sheets/ | 1 |
| /devops-vietnam-x-dataonline-uu-dai-vps-gia-re-hang-dau/ | 1 |
| /docs/ | 1 |
| /ebooks/ | 1 |
| /events/ | 1 |
| /flashcards/ | 1 |
| /forgot-password/ | 1 |
| /hehehe/ | 1 |
| /knowledge/ | 1 |
| /login/ | 1 |
| /mini-game/ | 1 |
| /partner/ | 1 |
| /privacy/ | 1 |
| /report/ | 1 |
| /services/ | 1 |
| /signup/ | 1 |
| /state-of-devops-vietnam-2026/ | 1 |
| /su-kien-devops/ | 1 |
| /support/ | 1 |
| /terms/ | 1 |
| /tin-tuc-devops/ | 1 |
| /0217/ | 1 |
| /category/ | 1 |
| /d/ | 1 |

Content-scope extraction: `body` = 2832, `dv-post-content` = 1033

## 3. Sitemap check

The site publishes `https://devops.vn/sitemap_index.xml` (Rank Math), which references 10 sub-sitemaps: `post-sitemap1..5`, `page-sitemap`, `series-sitemap`, `series_group-sitemap`, `author-sitemap`, `news-sitemap`.

All **1044** unique URLs from these sitemaps were extracted and crawled, with **1** exception(s):

- `https://devops.vn/account/` — auth-gated account page, redirects in a loop (login required); not public content.

Beyond the sitemap, a recursive link-following pass (BFS) crawled every additional internal page reachable via navigation and in-content links — taxonomy archives (`/tag/`), date archives (`/YYYY/MM/DD/`), author & user profiles (`/author/`, `/user/`), event archives (`/archived-events/`), category/series indexes, and their pagination — until no new page URLs were discovered (closure reached).

## 4. Failed pages (after retry)

Each failure was retried with a fresh request. All remain permanently unavailable — they are broken or non-existent links on the source site (dead internal links, old URL formats now returning 404, or access-restricted pages), **not** pages missed by the crawler. The real content they once pointed to (where any) lives under `/posts/…` and was crawled.

Breakdown: 1× TooManyRedirects, 36× 404, 1× 403

| URL | HTTP status |
|---|---|
| https://devops.vn/resources | 403 |
| https://devops.vn/blog | 404 |
| https://devops.vn/cach-cai-dat-docker-phien-ban-moi-nhat-tu-dong-bang-bashscript/ | 404 |
| https://devops.vn/cai-dat-aws-cli-huong-dan-chi-tiet/ | 404 |
| https://devops.vn/case-studies | 404 |
| https://devops.vn/cai-dat-kubernetes-tren-ubuntu-bang-kubeadm/ | 404 |
| https://devops.vn/category/cloud | 404 |
| https://devops.vn/category/devops | 404 |
| https://devops.vn/category/devops/ | 404 |
| https://devops.vn/ci-cd | 404 |
| https://devops.vn/new/chinh-thuc-ra-mat-viettel-container-registry/viettelidc.com.vn/viettel-container-registry | 404 |
| https://devops.vn/grafana | 404 |
| https://devops.vn/gitlab | 404 |
| https://devops.vn/posts/bai-2-cai-dat-minikube-va-su-dung-kubectl-co-ban/brew.sh | 404 |
| https://devops.vn/oracle | 404 |
| https://devops.vn/posts/tao-website-bang-hosting-mien-phi-tron-doi-cua-dataonline/my.dataonline.vn | 404 |
| https://devops.vn/resources/newsletters/ | 404 |
| https://devops.vn/series/ai-startup-discovery-day-2025/ | 404 |
| https://devops.vn/security | 404 |
| https://devops.vn/series/be-phong-cong-nghe-toan-dien-cho-ai-startup-viet-fpt-startup-innovation-2025-chinh-thuc-khoi-dong/ | 404 |
| https://devops.vn/series/biztech-2025-gap-go-dien-gia-tu-zoho/ | 404 |
| https://devops.vn/series/fpt-vietnam-tech-impact-summit-2024/ | 404 |
| https://devops.vn/series/hackathon-2025-good-and-evil-mixed-together/ | 404 |
| https://devops.vn/series/thu-thach-ai-trong-ngan-hang-techcombank-hack-together-2025/ | 404 |
| https://devops.vn/tag/CVE-2025-37164 | 404 |
| https://devops.vn/series/xplorators-2025-into-the-fintech-realms/ | 404 |
| https://devops.vn/tag/android/ | 404 |
| https://devops.vn/tag/appsec | 404 |
| https://devops.vn/tag/arm | 404 |
| https://devops.vn/tag/command-injection | 404 |
| https://devops.vn/tag/cline | 404 |
| https://devops.vn/tag/fortios | 404 |
| https://devops.vn/tag/fuzzing | 404 |
| https://devops.vn/tag/ios/ | 404 |
| https://devops.vn/tag/jenkinsx | 404 |
| https://devops.vn/tag/proxy | 404 |
| https://devops.vn/tag/vng | 404 |
| https://devops.vn/account/ | TooManyRedirects |

## 5. What each page file contains

Every `NNNN.md` file preserves the page structure without summarising:

- URL, Title, OG title, meta description, H1
- Full heading outline (H1–H6, nested)
- Full body text in structural markdown (paragraphs, lists, blockquotes)
- Highlight text (bold / italic / mark) and blockquotes, listed
- Images: URL + alt text
- Code blocks (fenced, language-tagged where available)
- Tables (markdown)
- Internal links and external links (separated)

See `INDEX.md` for the full file → URL mapping.
