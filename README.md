
# Install required packages
!pip install PyPDF2 TTS

import time
import PyPDF2
from TTS.api import TTS
from google.colab import files
from IPython.display import Audio, display

# Step 1: Upload the PDF file
print("Please upload your PDF file...")
uploaded = files.upload()
pdf_path = list(uploaded.keys())[0]

# Initialize the Coqui TTS model
tts = TTS(model_name="tts_models/en/ljspeech/tacotron2-DDC", progress_bar=False)

# Function to extract text from a single page of the PDF, limited to 300 words for faster processing
def extract_text_from_page(pdf_reader, page_num, max_words=300):
    page = pdf_reader.pages[page_num]
    text = page.extract_text()
    words = text.split()
    return ' '.join(words[:max_words])

# Convert a single page's text to speech, save, display audio, and wait for download to finish
def text_to_speech_single_page(text, page_num):
    filename = f"page_{page_num + 1}.mp3"
    tts.tts_to_file(text=text, file_path=filename)
    print(f"Audio file for page {page_num + 1} generated successfully.")

    # Preview the audio in the notebook
    display(Audio(filename))
    # Initiate download for each audio file immediately
    files.download(filename)

    # Wait for download to complete before proceeding
    download_confirm = input("Press Enter after downloading the file to proceed to the next page...")
    return filename

# Display estimated time per page and real-time progress
def display_progress(current_page, total_pages, time_taken):
    avg_time = time_taken / current_page if current_page > 0 else time_taken
    remaining_time = avg_time * (total_pages - current_page)
    print(f"\nProcessed page {current_page} of {total_pages}")
    print(f"Estimated remaining time: {int(remaining_time)} seconds\n")

# Main function to process the PDF one page at a time
def process_pdf_one_page_at_a_time(pdf_path):
    start_time = time.time()
    pdf_reader = PyPDF2.PdfReader(pdf_path)
    num_pages = len(pdf_reader.pages)
    
    for page_num in range(num_pages):
        # Display processing time estimate
        print(f"Processing page {page_num + 1} of {num_pages}...")
        page_start = time.time()

        # Extract text for the current page with a 300-word limit
        text = extract_text_from_page(pdf_reader, page_num)
        
        # Convert to speech, preview audio, and download
        if text.strip():
            filename = text_to_speech_single_page(text, page_num)
        else:
            print(f"No text found on page {page_num + 1}. Skipping...")
        
        # Update and display progress
        page_end = time.time()
        display_progress(page_num + 1, num_pages, page_end - start_time)
        time.sleep(0.5)  # Adding a short delay for smooth progress display

    print("\nAll pages have been processed successfully.")

# Run the main function
process_pdf_one_page_at_a_time(pdf_path)
