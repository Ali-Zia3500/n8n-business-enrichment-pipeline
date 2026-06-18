# n8n-business-enrichment-pipeline

# Automated Business Data Enrichment Pipeline

[![N8N](https://img.shields.io/badge/Built%20with-n8n-%23EA4B71.svg)](https://n8n.io/)
[![Apify](https://img.shields.io/badge/Powered%20by-Apify-%2300A1E0.svg)](https://apify.com/)
[![Supabase](https://img.shields.io/badge/Database-Supabase-%233FCF8E.svg)](https://supabase.com/)

An intelligent, multi-stage automation workflow built with **n8n** that discovers business leads via Google Maps, enriches them by extracting missing social media links (Facebook, Instagram, YouTube) and email addresses, and stores the enriched data in a Supabase database. The system employs a robust fallback strategy to maximize data collection from websites and Google Search results.

## 🚀 Key Features

- **Automated Lead Discovery**: Scrapes Google Maps based on business type and location to generate a list of potential leads.
- **Multi-Stage Data Enrichment**:
    1.  **Direct Website Scraping**: Attempts to extract social links and emails directly from the business' website.
    2.  **Google Search Fallback**: If direct scraping fails, it performs a targeted Google search using the business name and keywords to find social profiles.
- **Intelligent Data Parsing**: Uses custom JavaScript logic to:
    - Filter out junk/irrelevant URLs (e.g., `sharer`, `login`, `about` pages).
    - Extract clean, direct profile URLs for Facebook, Instagram, and YouTube.
    - Parse and validate email addresses from website HTML and search result snippets.
- **Data Preservation & Error Handling**: Designed to prevent overwriting existing, valid data in the database. If new data is found, it updates the record; if nothing new is found, it skips the update.
- **Batch Processing**: Processes large lists of businesses efficiently using n8n's batching capabilities.

## 🧠 How It Works

This workflow is triggered manually via an n8n **Form Trigger**, where a user inputs a "Business Type" and "Location."

1.  **Scrape Google Maps**: An Apify actor (`compass/crawler-google-places`) is called to discover businesses matching the criteria. It scrapes details like name, website, phone, address, and any existing social links.
2.  **Store Initial Data**: The discovered leads are formatted and inserted into a `businesses` table in **Supabase** with a status of `'ready_for_scraping'`.
3.  **Iterate & Enrich**: The workflow loops over each business record that is missing key data (e.g., `facebookurl`, `instagramurl`, `youtubeurl`, `email`).
4.  **Enrichment Pipeline**:
    - **Has Website?** The workflow checks if the business has a `website` URL.
    - **Branch 1 (Has Website):**
        - **Scrape Website**: Uses the **Apify Website Content Crawler** to fetch the HTML of the business' website.
        - **Extract Social Links**: A JavaScript node parses the HTML to extract Facebook, Instagram, and YouTube URLs.
        - **Found Links?** If links are found, it proceeds to update the database. If not, it enters a **fallback** stage.
        - **Fallback (Google Search)**: Runs an **Apify Google Search Scraper** with the query `"{business name} facebook instagram youtube Pakistan"`.
        - **Extract Fallback Links**: Another JavaScript node parses the search results to find and extract social media URLs.
    - **Branch 2 (No Website):**
        - Directly uses the **Apify Google Search Scraper** to find social links based on the business name alone.
5.  **Prepare for Update**: A JavaScript node intelligently merges newly found data with existing data. It only updates fields that have a new, non-empty value, preserving older valid data.
6.  **Update Database**: The final enriched data is updated in the corresponding Supabase record using the `placeid` as a unique identifier.
7.  **Logging**: Each processed record is logged to the console for monitoring.

## 🛠️ Tech Stack

- **Workflow Orchestration**: [n8n](https://n8n.io/)
- **Data Storage**: [Supabase](https://supabase.com/) (PostgreSQL)
- **Web Scraping & Search**: [Apify](https://apify.com/) (Actors for Google Maps, Website Crawler, Google Search)
- **Backend Logic**: n8n's built-in Code/JavaScript nodes
- **Deployment**: Self-hosted n8n instance

## 📋 Prerequisites

To run this workflow, you need:

1.  **n8n Instance**: A self-hosted or cloud n8n instance (version 1.0+).
2.  **Apify Account**: An Apify account with API token for running actors.
3.  **Supabase Project**: A Supabase project with a `businesses` table.
4.  **Credentials**: API keys and tokens for Apify and Supabase configured in your n8n instance.

## ⚙️ Setup & Configuration

1.  **Import Workflow**: Copy the JSON code of the workflow and paste it into a new workflow in your n8n instance.
2.  **Configure Credentials**:
    - **Supabase**: Add your Supabase URL and Service Role Key.
    - **Apify**: Add your Apify API token.
3.  **Set Up Supabase Table**: Create a `businesses` table in your Supabase database. Ensure it includes the following fields (data types are suggestions):
    - `id` (int8, primary key)
    - `name` (text)
    - `placeid` (text, unique)
    - `website` (text)
    - `facebookurl` (text)
    - `instagramurl` (text)
    - `youtubeurl` (text)
    - `email` (text)
    - `phone` (text)
    - `address` (text)
    - `status` (text)
    - `googlereviewsurl` (text)
4.  **Save & Activate**: Save the workflow. The workflow is triggered manually, so you can click "Execute Workflow" after filling in the form.

## 🔧 Workflow Customization

- **Adjust Search Parameters**: You can modify the `customBody` in the "Scrape Google Maps" node to change the number of results (`maxCrawledPlacesPerSearch`) or add more search strings.
- **Update Junk Filters**: The JavaScript nodes contain lists of junk URL segments (e.g., `FB_JUNK`, `IG_JUNK`). You can add or remove terms based on your needs.
- **Change the Database Table**: The "Get Businesses Without Social Links" node uses a filter on the `facebookurl`, `instagramurl`, `youtubeurl`, and `email` fields. You can modify this to target different conditions.

## 🐛 Error Handling & Logging

- The workflow is designed to continue on error for most Apify and HTTP nodes. This ensures that if one business fails, the entire pipeline does not stop.
- All processed business results are logged to the n8n console, making it easy to track which records were updated.

## 🤝 Contributing

Contributions are welcome! If you have suggestions for improvements or bug fixes, please feel free to open an issue or submit a pull request.

## 📄 License

This project is open-source and available under the [MIT License](LICENSE).

## 👤 Author

**Ali Zia**

- GitHub: [@Ali-Zia3500](https://github.com/Ali-Zia3500)

---

### ⭐ If you find this project useful, please consider giving it a star on GitHub!
