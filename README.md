import requests
from bs4 import BeautifulSoup
import csv
import time


BASE_URL = "https://quotes.toscrape.com"
START_URL = BASE_URL + "/"
OUTPUT_FILE = "quotes.csv"


def scrape_quotes():
    """
    Scrapes quotes, authors and tags from Quotes to Scrape.
    Handles pagination and saves the result into a CSV file.
    """

    all_quotes = []
    page_url = START_URL
    page_number = 1

    headers = {
        "User-Agent": (
            "Mozilla/5.0 (Windows NT 10.0; Win64; x64) "
            "AppleWebKit/537.36 (KHTML, like Gecko) "
            "Chrome/120.0 Safari/537.36"
        )
    }

    while page_url:

        print(f"Scraping page {page_number}...")

        try:
            response = requests.get(
                page_url,
                headers=headers,
                timeout=10
            )

            response.raise_for_status()

        except requests.RequestException as error:
            print(f"Error while requesting page: {error}")
            break

        soup = BeautifulSoup(response.text, "html.parser")

        quotes = soup.find_all("div", class_="quote")

        if not quotes:
            print("No more quotes found.")
            break

        for quote in quotes:

            # Extract quote text
            text_element = quote.find("span", class_="text")

            # Extract author
            author_element = quote.find("small", class_="author")

            # Extract tags
            tag_elements = quote.find_all("a", class_="tag")

            quote_text = (
                text_element.get_text(strip=True)
                if text_element
                else ""
            )

            author = (
                author_element.get_text(strip=True)
                if author_element
                else ""
            )

            tags = ", ".join(
                tag.get_text(strip=True)
                for tag in tag_elements
            )

            all_quotes.append({
                "Quote": quote_text,
                "Author": author,
                "Tags": tags,
                "Page": page_number
            })

        # Find next page
        next_button = soup.find(
            "li",
            class_="next"
        )

        if next_button:

            next_link = next_button.find("a")

            if next_link and next_link.get("href"):

                next_page = next_link["href"]

                page_url = BASE_URL + next_page

                page_number += 1

                # Small delay between requests
                time.sleep(1)

            else:
                page_url = None

        else:
            page_url = None

    return all_quotes


def save_to_csv(data):
    """
    Saves scraped data into a CSV file.
    """

    if not data:
        print("No data available to save.")
        return

    fieldnames = [
        "Quote",
        "Author",
        "Tags",
        "Page"
    ]

    try:
        with open(
            OUTPUT_FILE,
            "w",
            newline="",
            encoding="utf-8"
        ) as file:

            writer = csv.DictWriter(
                file,
                fieldnames=fieldnames
            )

            writer.writeheader()
            writer.writerows(data)

        print(f"\nData successfully saved to {OUTPUT_FILE}")

    except IOError as error:
        print(f"Error while saving CSV file: {error}")


def display_summary(data):
    """
    Displays basic information about the scraped dataset.
    """

    if not data:
        return

    authors = set(
        item["Author"]
        for item in data
        if item["Author"]
    )

    tags = set()

    for item in data:

        if item["Tags"]:

            for tag in item["Tags"].split(","):
                tags.add(tag.strip())

    print("\n" + "=" * 50)
    print("SCRAPING SUMMARY")
    print("=" * 50)

    print(f"Total quotes scraped : {len(data)}")
    print(f"Unique authors      : {len(authors)}")
    print(f"Unique tags         : {len(tags)}")

    print("\nFirst 5 records:")

    for item in data[:5]:

        print(
            f"\nQuote  : {item['Quote']}"
            f"\nAuthor : {item['Author']}"
            f"\nTags   : {item['Tags']}"
        )


def main():

    print("=" * 50)
    print("      WEB SCRAPING PROJECT")
    print("=" * 50)

    print("\nStarting scraper...\n")

    scraped_data = scrape_quotes()

    save_to_csv(scraped_data)

    display_summary(scraped_data)

    print("\nScraping completed successfully!")


if __name__ == "__main__":
    main()
