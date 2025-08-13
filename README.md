import random
import string

print("=== Password Generator ===")

# Step 1: Ask user for length
length = int(input("Enter the length of the password: "))

# Step 2: Prepare characters
lowercase = string.ascii_lowercase
uppercase = string.ascii_uppercase
numbers = string.digits
symbols = "!@#$%^&*()-_=+[]{};:,.<>?/\\|"

# Step 3: Combine all characters
all_characters = lowercase + uppercase + numbers + symbols

# Step 4: Make sure password has unique characters
password = ""
used_chars = set()

while len(password) < length:
    char = random.choice(all_characters)
    if char not in used_chars:   # avoid duplicates
        password += char
        used_chars.add(char)

# Step 5: Show password
print("Your Generated Password:", password)
