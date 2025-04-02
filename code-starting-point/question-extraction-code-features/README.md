# Question  Extraction Code Features

The updated code processes a PDF file containing SAT Math questions, extracts relevant information from each page, and outputs the data into a JSON file. The latest version includes a new feature to assign Item Response Theory (IRT) 3PL model parameters ((a), (b), (c)) to each question based on its difficulty level and question type. Below, I'll explain the key features of the code, including the new IRT 3PL parameter assignment, and provide a detailed breakdown of its functionality.

***

#### Key Features of the Latest Code

1. **PDF to Image Conversion**:
   * The code uses the `pdf2image` library (`convert_from_path`) to convert each page of the input PDF into an image for processing.
   * It processes a specified range of pages (e.g., `start_page` to `end_page`), allowing flexibility in handling large documents.
2. **Image Processing with OpenCV**:
   * The code uses OpenCV (`cv2`) to perform image processing tasks, such as detecting graphs, tables, lines, curves, and scatter points.
   * Images are converted from RGB (from `pdf2image`) to BGR format for compatibility with OpenCV functions.
3. **Text Extraction with OCR**:
   * The `pytesseract` library is used to extract text from each page image via Optical Character Recognition (OCR).
   * The `extract_text` function allows excluding specific regions (e.g., graphs or tables) to improve text extraction accuracy.
4. **Graph and Table Detection**:
   * The `detect_graph_or_table` function identifies whether a page contains a graph or table by analyzing contours and edges in the image.
   * It uses helper functions like `detect_lines`, `detect_curves`, and `detect_scatter_points` to classify regions as "graph" or "table":
     * **Lines**: Detected using the Hough Line Transform (`cv2.HoughLinesP`).
     * **Curves**: Identified by analyzing contours with more than 4 sides (non-rectangular shapes).
     * **Scatter Points**: Detected as small, isolated contours (area between 10 and 100 pixels).
   * If a graph or table is detected, its image is converted to a base64 string (via `image_to_base64`) and stored in the JSON output along with its boundaries.
5. **Equation Detection**:
   * The `detect_equations` function identifies mathematical equations in the extracted text using two methods:
     * **LaTeX Patterns**: Matches LaTeX equations enclosed in `$...$` or `$$...$$`.
     * **Math Symbols**: Identifies inline equations containing symbols like `+`, `-`, `*`, `/`, `=`, or `^`.
   * Equations are stored as LaTeX strings in the JSON output under the `equations` field.
6. **Metadata Parsing**:
   * The `parse_metadata` function extracts metadata from the text, including:
     * **Question ID**: Using a regex pattern to match "Question ID" followed by a hexadecimal string.
     * **Assessment Details**: Parsed from a table-like structure (delimited by `|`) containing fields like Assessment, Test, Domain, Skill, and Difficulty.
   * This metadata is stored in the `metadata` field of the JSON output.
7. **Question and Answer Extraction**:
   * The `process_pdf_to_json` function processes the extracted text to separate the question, answer choices, correct answer, and rationale:
     * **Question Text**: Lines before the "Correct Answer:" marker, excluding metadata and answer choices.
     * **Answer Choices**: Extracted using `extract_answer_choices`, which identifies lines starting with A, B, C, or D (e.g., "A.", "B)", etc.) and validates that MCQs have exactly 4 options.
     * **Correct Answer**: Identified after the "Correct Answer:" marker.
     * **Rationale**: Text following the correct answer, explaining the solution.
   * The question type is determined as "MCQ" (Multiple Choice Question) if answer choices are present, or "FRQ" (Free Response Question) if not.
8. **IRT 3PL Parameter Assignment (New Feature)**:
   * The `assign_irt_parameters` function assigns IRT 3PL parameters ((a), (b), (c)) to each question based on its difficulty and type:
     * **Discrimination ((a))**: Measures how well the item differentiates between test-takers of different abilities.
       * Easy: (a = 0.8)
       * Medium: (a = 1.0)
       * Hard: (a = 1.2)
     * **Difficulty ((b))**: The ability level ((\theta)) at which a test-taker has a 50% chance of answering correctly (adjusted for guessing).
       * Easy: (b = -1.0)
       * Medium: (b = 0.0)
       * Hard: (b = 1.0)
     * **Guessing ((c))**: The probability of a low-ability test-taker answering correctly by guessing.
       * MCQ: (c = 0.25) (for 4 options)
       * FRQ: (c = 0.0) (no guessing for grid-in questions)
   * These parameters are added to the JSON output under the `irt_parameters` field for each question.
9. **JSON Output**:
   * The `process_pdf_to_json` function compiles all extracted data into a JSON file with the following structure for each question:
     * `page`: Page number.
     * `metadata`: Question ID, Assessment, Test, Domain, Skill, Difficulty.
     * `question`: The question text.
     * `graph_or_table`: Details of any detected graph or table (type, base64 image, boundaries).
     * `equations`: List of detected LaTeX equations.
     * `question_type`: "MCQ" or "FRQ".
     * `answer_choices`: List of answer choices (for MCQs).
     * `correct_answer`: The correct answer.
     * `rationale`: Explanation of the solution.
     * `irt_parameters`: IRT 3PL parameters ((a), (b), (c)).
   * The JSON file is saved to the specified `output_json_path` with proper indentation.
10. **Error Handling and Logging**:
    * The code uses Python's `logging` module to log informational messages, warnings, and errors.
    * It includes validation for MCQs (ensuring exactly 4 answer choices) and attempts to recover missing choices if possible.
    * Exceptions during processing are caught and reported in the main block.

***

#### Detailed Breakdown of the IRT 3PL Parameter Assignment

The IRT 3PL (Three-Parameter Logistic) model is a psychometric model used to describe the relationship between a test-taker's ability ((\theta)) and the probability of answering an item correctly. The probability is given by:

\[\
P(\theta) = c + (1 - c) \frac{1}{1 + e^{-a(\theta - b)\}}\
]

* **(a) (Discrimination)**: Controls the steepness of the item characteristic curve (ICC). Higher values mean the item better distinguishes between test-takers of different abilities.
* **(b) (Difficulty)**: The ability level at which the probability of a correct response is 50% (adjusted for guessing). It’s on a standard normal scale (mean 0, SD 1).
* **(c) (Guessing)**: The lower asymptote of the ICC, representing the probability of a correct response for a test-taker with very low ability.

**Assignment Logic**:

* The `assign_irt_parameters` function uses the difficulty level from the metadata and the question type to assign parameters:
  * **Difficulty Mapping**:
    * Easy ((b = -1.0)): Test-takers with below-average ability ((\theta < 0)) have a 50% chance of answering correctly.
    * Medium ((b = 0.0)): Test-takers with average ability ((\theta = 0)) have a 50% chance.
    * Hard ((b = 1.0)): Test-takers with above-average ability ((\theta > 0)) have a 50% chance.
  * **Discrimination Mapping**:
    * Easy ((a = 0.8)): Less discriminating, as easy items are often less sensitive to ability differences.
    * Medium ((a = 1.0)): Typical discrimination for many items.
    * Hard ((a = 1.2)): More discriminating, as hard items better differentiate high-ability test-takers.
  * **Guessing Parameter**:
    * For MCQs, (c = 0.25), reflecting a 25% chance of guessing correctly with 4 options.
    * For FRQs (grid-in questions), (c = 0.0), as guessing is unlikely to yield the correct answer.

**Integration**:

* The difficulty is extracted from the `metadata` field of each question.
* The question type is determined by the presence of answer choices ("MCQ" if choices exist, "FRQ" if not).
* The `irt_parameters` field is added to the JSON output, making the IRT parameters available for further analysis (e.g., test calibration, ability estimation).

***

#### Example Workflow for a Single Page

1. **Input**: A PDF page (e.g., page 36) with a question, metadata, and possibly a graph.
2. **Image Conversion**: The page is converted to an image using `convert_from_path`.
3. **Graph/Table Detection**:
   * `detect_graph_or_table` checks for graphs or tables.
   * If found, the region is converted to base64 and excluded from text extraction.
4. **Text Extraction**: `extract_text` uses `pytesseract` to extract text from the image.
5. **Metadata Parsing**: `parse_metadata` extracts the question ID, assessment details, and difficulty (e.g., "Easy").
6. **Equation Detection**: `detect_equations` identifies LaTeX equations in the text.
7. **Question and Answer Extraction**:
   * The question text, answer choices, correct answer, and rationale are separated.
   * `extract_answer_choices` identifies MCQ options (if any).
8. **Question Type Determination**: Set to "MCQ" if answer choices are present, "FRQ" otherwise.
9. **IRT Parameter Assignment**:
   * Difficulty = "Easy", Question Type = "FRQ".
   * `assign_irt_parameters` assigns: (a = 0.8), (b = -1.0), (c = 0.0).
10. **JSON Output**: The result is added to the `results` list with all fields, including `irt_parameters`.

***

#### Benefits of the Latest Code

* **Comprehensive Data Extraction**: Extracts all relevant components of a question (text, equations, graphs/tables, answers, rationale).
* **IRT 3PL Integration**: Adds psychometric parameters to each question, enabling advanced analysis like test calibration or adaptive testing.
* **Robust Error Handling**: Includes logging and validation to handle issues like missing answer choices or invalid metadata.
* **Flexible Processing**: Allows processing a specific range of pages, making it efficient for large PDFs.
* **Structured Output**: The JSON format is well-organized and suitable for downstream applications (e.g., educational platforms, test analysis tools).

***

#### Limitations and Potential Improvements

1. **Static IRT Parameters**:
   * The IRT parameters are assigned based on predefined rules. In practice, these parameters should be estimated using response data (e.g., via maximum likelihood estimation).
   * **Improvement**: Integrate a library like `mirt` (in R) or `pystan` to estimate IRT parameters if response data is available.
2. **OCR Accuracy**:
   * The accuracy of text extraction depends on `pytesseract`. Complex layouts or poor image quality may lead to errors.
   * **Improvement**: Preprocess images (e.g., enhance contrast, remove noise) or use a more advanced OCR engine like Google Cloud Vision API.
3. **Equation Detection**:
   * The current method may miss some equations or include false positives.
   * **Improvement**: Use a dedicated math OCR tool like MathPix to improve equation detection and parsing.
4. **Graph/Table Handling**:
   * The detection of graphs and tables is heuristic-based and may misclassify complex figures.
   * **Improvement**: Use machine learning models (e.g., a CNN) to classify regions more accurately.
5. **Scalability**:
   * Processing large PDFs with many pages can be slow due to image conversion and OCR.
   * **Improvement**: Parallelize the processing of pages using Python's `multiprocessing` module.

***

#### Conclusion

The latest code is a robust tool for extracting SAT Math questions from a PDF and structuring them into a JSON format, with the added functionality of assigning IRT 3PL parameters. It supports a wide range of question types, handles visual elements like graphs and tables, and provides detailed metadata and rationales. The IRT 3PL parameters enable psychometric analysis, making the output suitable for educational research, test development, or adaptive learning systems. With some potential improvements, the code can be made even more accurate and scalable for real-world applications.
