[opengraph]

OpenGraph — how much does it matter for SEO?

Important distinction: OpenGraph tags have ~zero effect on Google ranking or crawling. They're consumed by social platforms (Facebook, LinkedIn, WhatsApp, Slack, Twitter/X) when generating a link preview card — title, description, image shown when someone shares a wework.com URL. Google does not use og:title/og:description as ranking or snippet signals (it has its own logic based on <title>/meta description/page content).

So "almost no OG data in prod" is not an SEO gap — it's a social-sharing/CTR gap. The practical cost is: when someone shares a page link in WhatsApp/LinkedIn/Slack, the preview falls back to defaults (fallbackTitle = metaTitle, no image) or renders a blank/ugly card instead of a branded image + tailored description. That hurts click-through on shared links, not search rank. Worth fixing for pages that get shared a lot (blog posts, campaign landing pages), but it's a separate concern from metaTitle/description/schemaData, which are the fields that actually move search visibility.