import asyncio
from playwright.async_api import async_playwright
import json

async def scrape_yelp_reviews(business_url, max_pages=3):
    reviews = []

    async with async_playwright() as p:
        browser = await p.chromium.launch(headless=True)
        context = await browser.new_context()
        page = await context.new_page()

        # Intercept network responses
        async def handle_response(response):
            if "review_feed" in response.url:
                try:
                    json_data = await response.json()
                    # Yelp review data lives under 'reviews' key usually
                    for review in json_data.get("reviews", []):
                        text = review.get("comment", {}).get("text", "").strip()
                        date = review.get("localizedDate", "")
                        if text:
                            reviews.append((text, date))
                except Exception as e:
                    print(f"Error parsing JSON: {e}")

        page.on("response", handle_response)

        # Go to the business page sorted by newest reviews
        await page.goto(business_url)

        # Wait enough time for network requests and responses
        await asyncio.sleep(5)

        # Yelp paginates reviews with a 'start' query param in API calls,
        # so we can navigate pages by changing URL with &start=X

        for i in range(1, max_pages):
            next_page_url = f"{business_url}&start={i * 20}"
            await page.goto(next_page_url)
            await asyncio.sleep(5)  # wait for API calls on each page

        await browser.close()

    return reviews

# Example usage:
business_url = "https://www.yelp.com/biz/coffee-and-chisme-south-gate?sort_by=date_desc"

async def main():
    reviews = await scrape_yelp_reviews(business_url, max_pages=3)
    for i, (text, date) in enumerate(reviews, 1):
        print(f"{i}. [{date}] {text}\n")

asyncio.run(main())
