# PANTHER
Categorization &amp; Sentiment Model.
---

# Enhanced Sentiment and Text Categorization Engine

## Overview

This project provides a sophisticated Python-based engine for performing advanced text analysis on user feedback, such as survey responses or reviews. It goes beyond simple keyword matching by implementing a hybrid model that combines Net Promoter Score (NPS) with a contextual semantic analysis of the accompanying text.

The primary goal is to classify feedback into specific business-relevant categories (e.g., "Customer Service," "Product Quality," "Pricing") and assign a nuanced sentiment score ("positive," "negative," "neutral") that considers negations, intensifiers, and the initial NPS rating.

A key feature is its **two-pass categorization strategy**, designed to minimize unclassified responses and improve accuracy by first assigning specific topics and then using a general "satisfaction" category for comments that were not initially classified.

## Key Features

-   **Context-Aware Sentiment Analysis**: The engine understands context. It correctly interprets phrases like "not good" (negation) and "very good" (intensifier) instead of just counting positive/negative words.
-   **Hybrid NPS + Text Model**: It leverages the quantitative NPS score as a baseline for sentiment and then refines it based on the semantic content of the written feedback. This creates a more accurate and holistic sentiment score.
-   **Two-Pass Hierarchical Categorization**: To improve classification accuracy, the system performs two passes:
    1.  **First Pass**: Assigns comments to specific, detailed categories.
    2.  **Second Pass**: Re-evaluates any unclassified comments from the first pass against a broader, more general "Satisfaction" category. This ensures that generic positive or negative feedback is still captured without overriding more specific topics.
-   **Advanced Text Preprocessing**: Includes robust text cleaning steps such as lowercasing, accent removal, punctuation stripping, stopword removal, and lemmatization (reducing words to their root form) for more accurate matching.
-   **Customizable and Scalable**: The entire logic is driven by external JSON configuration files, allowing you to easily update keywords, categories, sentiment weights, and language rules without changing the source code.
-   **Efficient Processing**: Utilizes `pandas` for data manipulation and `swifter` to apply text processing functions in parallel, speeding up execution on large datasets.

## How It Works

The processing pipeline is executed as follows:

1.  **Configuration Loading**: The engine loads all rules, keywords, weights, and parameters from two main JSON files:
    *   `categorias_v2.json`: Defines the thematic categories, their associated keywords, and their weights.
    *   `sentimientos_v2.json`: Contains sentiment lexicons (positive/negative words), negation words, intensifiers, special phrases, and the parameters for the sentiment scoring algorithm.

2.  **Data Loading and Cleaning**: It loads the input data from a CSV file, standardizes column names, and performs a comprehensive text cleaning and lemmatization process on the feedback column.

3.  **Categorization (Two-Pass Strategy)**:
    *   The `procesar_dataframe_eficiente` function first instantiates an analyzer *without* the general 'Extras_Satisfaccion' category and classifies all comments.
    *   It then identifies all comments that were marked as `no_clasificado`.
    *   A second, lightweight analyzer is instantiated *only* with the 'Extras_Satisfaccion' category. This analyzer processes only the unclassified comments.
    *   The results are merged, significantly reducing the number of unclassified entries.

4.  **Sentiment Analysis**:
    *   After categorization is complete, the `analizar_sentimiento` method is called for every record.
    *   It determines a baseline score from the NPS rating (promoter, passive, or detractor).
    *   It then analyzes the cleaned text, adjusting the score based on positive/negative words, intensifiers (e.g., "very"), and negations (e.g., "not").
    *   A final sentiment label (`positive`, `negative`, `neutral`) is assigned based on configurable thresholds. Special rules are applied for edge cases (e.g., a promoter with negative text).

5.  **Output Generation**: The final `DataFrame`, enriched with columns for categories, sentiment, scores, and cleaned text, is saved as a new CSV file in the `bases_categorizadas` directory.

## Project Structure

```
.
├── DIRECTORIO_PROYECTO/
│   └── base_a_procesar.csv       # Input data file
├── diccionarios/
│   ├── categorias_v2.json        # Configuration for topics and keywords
│   └── sentimientos_v2.json       # Configuration for sentiment analysis
├── bases_categorizadas/
│   └── base_a_procesar_categorizada_v2.csv # Output file
└── main_script.py                  # The main executable Python script
```

-   **`main_script.py`**: The core script that orchestrates the entire process.
-   **`diccionarios/`**: This directory holds the JSON configuration files that control the engine's behavior.
-   **`DIRECTORIO_PROYECTO/`**: The designated folder for your input CSV data.
-   **`bases_categorizadas/`**: The default output directory where processed files are saved.

## Getting Started

### Prerequisites

Make sure you have Python 3.8+ installed. Then, install the required libraries:

```bash
pip install pandas numpy swifter unidecode tqdm
```

### Configuration

1.  **Input Data**: Place your input CSV file in the `DIRECTORIO_PROYECTO` folder. The script expects columns for an ID, an NPS score, and the text comment. You can map your column names inside the `main` function in the `COLUMN_MAPPING` dictionary.

2.  **Paths**: In the `if __name__ == "__main__":` block of the script, update the `ruta_base_proyecto` variable to point to the absolute path of your project's root folder.

3.  **Categories (`categorias_v2.json`)**:
    *   `"categorias"`: A dictionary where keys are category names and values are lists of keywords and phrases associated with that topic.
    *   `"pesos_categorias"`: Assign a weight to each category to make it more or less important during classification.

4.  **Sentiment (`sentimientos_v2.json`)**:
    *   `"sentimiento_positivo"` / `"sentimiento_negativo"`: Lists of positive and negative words.
    *   `"sentimiento_params"`: Fine-tune the sentiment algorithm with parameters like `negation_impact`, `intensifier_weights`, and scoring thresholds.

### Running the Analysis

Once configured, simply run the script from your terminal:

```bash
python main_script.py
```

The script will log its progress to the console and, upon completion, will print the location of the output CSV file.

## Dependencies

-   [pandas](https://pandas.pydata.org/): For data manipulation and analysis.
-   [numpy](https://numpy.org/): For numerical operations.
-   [swifter](https://github.com/jmcarpenter2/swifter): To efficiently parallelize pandas `apply` functions.
-   [unidecode](https://pypi.org/project/Unidecode/): For transliterating Unicode text into ASCII (e.g., removing accents).
-   [tqdm](https://github.com/tqdm/tqdm): For displaying smart progress bars.
