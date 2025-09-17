
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>EmployMe - Talent Management</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script>
      tailwind.config = {
        theme: {
          extend: {
            colors: {
              'brand-blue': '#264a8d',
              'brand-orange': '#f16528',
              'brand-blue-light': '#3b5a9d',
            }
          }
        }
      }
    </script>
<script type="importmap">
{
  "imports": {
    "react/": "https://aistudiocdn.com/react@^19.1.1/",
    "react": "https://aistudiocdn.com/react@^19.1.1",
    "react-dom/": "https://aistudiocdn.com/react-dom@^19.1.1/",
    "@google/genai": "https://aistudiocdn.com/@google/genai@^1.20.0"
  }
}
</script>
</head>
<body class="bg-gray-50">
    <div id="root"></div>
    <script type="module" src="/index.tsx"></script>
</body>
</html>
