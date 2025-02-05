#!/usr/bin/env sh

# Function to check for hardcoded URLs and form inputs in Playwright test and page files
check_hardcoded_inputs() {
  local file=$1
  local patterns=(
    "goto\(\s*['\"][^'\"]+['\"]\s*\)"
    "\.goto\(\s*['\"][^'\"]+['\"]\s*\)"
    "fill\(\s*['\"][^'\"]+['\"]\s*\)"
    "\.fill\(\s*['\"][^'\"]+['\"]\s*\)"
    "request\(\s*['\"][^'\"]+['\"]\s*\)"
    "\.request\(\s*['\"][^'\"]+['\"]\s*\)"
  )

  for pattern in "${patterns[@]}"; do
    local match=$(grep -E "$pattern" "$file")
    if [ -n "$match" ]; then
      echo "Hardcoded value found in file: $file"
      echo "$match"
      return 1
    fi
  done
  return 0
}

# Get a list of modified files
MODIFIED_FILES=$(git diff --cached --name-only --diff-filter=AM | grep -v "node_modules")

# Filter for Playwright test files in the tests folder
PLAYWRIGHT_FILES=$(echo "$MODIFIED_FILES" | grep -E '\.spec\.js$' | grep 'tests')

# Filter for JavaScript files in the pages folder
PAGE_FILES=$(echo "$MODIFIED_FILES" | grep -E '\.js$' | grep 'pages')

# Filter for files supported by Prettier (excluding node_modules)
PRETTIER_FILES=$(echo "$MODIFIED_FILES" | grep -E '\.(js|jsx|ts|tsx|json|css|scss|md)$')

# Exit early if no relevant files were modified
if [ -z "$PLAYWRIGHT_FILES" ] && [ -z "$PAGE_FILES" ] && [ -z "$PRETTIER_FILES" ]; then
  exit 0
fi

# Flag to track if there are any violations
VIOLATION_FOUND=0

# Check each modified Playwright test file and page file for hardcoded inputs
for file in $PLAYWRIGHT_FILES $PAGE_FILES; do
  check_hardcoded_inputs "$file" || VIOLATION_FOUND=1
done

# Run Prettier on the modified files
if [ -n "$PRETTIER_FILES" ]; then
  echo "Running Prettier to format files..."
  npx prettier --write $PRETTIER_FILES
  PRETTIER_EXIT_CODE=$?
  if [ $PRETTIER_EXIT_CODE -ne 0 ]; then
    echo "Prettier failed to format some files. Please fix them and try again."
    exit $PRETTIER_EXIT_CODE
  fi

  # Stage the formatted files
  echo "Staging formatted files..."
  echo "$PRETTIER_FILES" | xargs git add
fi

# Run Playwright tests on modified files in the tests folder
if [ -n "$PLAYWRIGHT_FILES" ]; then
  echo "Running Playwright tests on modified files..."
  for file in $PLAYWRIGHT_FILES; do
    npx playwright test "$file"
    PLAYWRIGHT_EXIT_CODE=$?
    if [ $PLAYWRIGHT_EXIT_CODE -ne 0 ]; then
      echo "Playwright tests failed. Please fix them and try again."
      exit $PLAYWRIGHT_EXIT_CODE
    fi
  done
fi

# If there are violations, exit with error code
if [ $VIOLATION_FOUND -ne 0 ]; then
  echo "Please use variables for all URLs and input values in Playwright tests and page files."
  exit 1
fi

# If everything is okay, exit with code 0
exit 0
