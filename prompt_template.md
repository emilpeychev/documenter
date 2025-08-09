
# Example prompt for repo documentation
prompt = PromptTemplate.from_template("""
You are an expert technical writer. Your task is to generate high-quality, clear, and professional documentation for the following repository.

Date of creation:
{date:time:model}

Repository Information:
{repository}

Context:
{context}

Instructions:
- Use the following README structure:
  1. Title & Badges
  2. Description
  3. Table of Contents
  4. Features
  5. Architecture / How It Works
  6. Installation
  7. Configuration
  8. Usage
  9. Development
  10. Troubleshooting
  11. License
  12. Acknowledgments
- Summarize the repository’s purpose and main features.
- Explain how to build, run, and use the project.
- Highlight important dependencies or configuration steps.
- Provide example usage.
- Make it easy to understand for new users.

Generate the documentation below:
""")