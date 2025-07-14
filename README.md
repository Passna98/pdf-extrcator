import os
import requests
from urllib.parse import urljoin
from bs4 import BeautifulSoup
import pandas as pd
import datetime

def extract_url_pdf(input_url, folder_location=os.getcwd()):
    # Create folder if it doesn't exist
    if not os.path.exists(folder_location):
        os.mkdir(folder_location)
        print(f"✅ Folder created at: {folder_location}")

    # Send GET request
    response = requests.get(input_url)
    soup = BeautifulSoup(response.text, "html.parser")

    # Lists for storing data
    link_text = []
    link_href = []
    link_files = []
    counter = 0

    # Find all PDF links
    for link in soup.select("a[href$='.pdf']"):
        filename = os.path.join(folder_location, link["href"].split("/")[-1])
        with open(filename, "wb") as f:
            f.write(requests.get(urljoin(input_url, link["href"])).content)

        # Save metadata
        link_text.append(link.text.strip())
        link_href.append(urljoin(input_url, link["href"]))
        link_files.append(link["href"].split("/")[-1])
        counter += 1
        print(f"{counter} - File downloaded: {link['href'].split('/')[-1]}")

    # Save metadata to Excel
    timestamp = datetime.datetime.now().strftime('%Y-%m-%d_%H-%M-%S')
    excel_output = os.path.join(folder_location, f"Excel_Output_{timestamp}.xlsx")
    writer = pd.ExcelWriter(excel_output, engine="openpyxl")
    df = pd.DataFrame({
        "Text": link_text,
        "Url_Link": link_href,
        "File_Name": link_files
    })
    df.to_excel(writer, sheet_name="Output", index=False)
    writer.close()

    print("\n✅ All PDFs downloaded and Excel file created at:")
    print(excel_output)

# Example usage
extract_url_pdf ")# pdf-extrcator
