# Password-Generator
# Imran's Pro Password Generator (interactive, original)
# Features:
# - Choose length and how many passwords to make
# - Toggle lowercase/UPPERCASE/digits/symbols
# - Option to avoid confusing characters like O/0, l/1, |, S/5, B/8
# - Ensures at least one of every selected type
# - Avoids triple repeats (like 'aaa' or '111')

import secrets
import string

def yes_no(prompt, default=True):
    d = "Y/n" if default else "y/N"
    while True:
        ans = input(f"{prompt} ({d}): ").strip().lower()
        if ans == "" and default is not None:
            return default
        if ans in ("y", "yes"): return True
        if ans in ("n", "no"):  return False
        print("Please answer with yes or no.")

def read_int(prompt, min_val, max_val=None):
    while True:
        try:
            n = int(input(prompt))
            if n < min_val:
                print(f"Enter a number ≥ {min_val}.")
                continue
            if max_val is not None and n > max_val:
                print(f"Enter a number ≤ {max_val}.")
                continue
            return n
        except ValueError:
            print("Please enter a valid integer.")

def secure_shuffle(chars_list):
    # Fisher–Yates using cryptographically secure randomness
    for i in range(len(chars_list)-1, 0, -1):
        j = secrets.randbelow(i + 1)
        chars_list[i], chars_list[j] = chars_list[j], chars_list[i]

def filter_ambiguous(s, avoid_confusing):
    if not avoid_confusing:
        return s
    ambiguous = "O0oIl1|S5B8"
    return "".join(ch for ch in s if ch not in ambiguous)

def build_pool(use_lower, use_upper, use_digits, use_symbols, avoid_confusing):
    pools = []
    if use_lower:  pools.append(filter_ambiguous(string.ascii_lowercase, avoid_confusing))
    if use_upper:  pools.append(filter_ambiguous(string.ascii_uppercase, avoid_confusing))
    if use_digits: pools.append(filter_ambiguous(string.digits,            avoid_confusing))
    if use_symbols:
        # Pick a safe subset of punctuation (drop whitespace-like or odd ones)
        base = "!@#$%^&*()-_=+[]{};:,.?/\\<>"
        pools.append(filter_ambiguous(base, avoid_confusing))
    # Merge to one pool string
    merged = "".join(pools)
    return pools, merged

def no_triples_ok(pw):
    # Disallow three identical in a row
    for i in range(len(pw) - 2):
        if pw[i] == pw[i+1] == pw[i+2]:
            return False
    return True

def generate_one(length, pools, merged):
    must_pick = []  # ensure at least one from each selected pool
    for p in pools:
        if p:  # pool might be empty if filtering removed everything
            must_pick.append(secrets.choice(p))
    if len(must_pick) > length:
        return None  # impossible with current length

    # Fill the rest from merged pool
    remaining = length - len(must_pick)
    body = [secrets.choice(merged) for _ in range(remaining)]
    candidate = must_pick + body
    secure_shuffle(candidate)

    # Enforce "no triple repeat" by resampling a few times if needed
    for _ in range(20):
        if no_triples_ok(candidate):
            return "".join(candidate)
        # mutate a random position using the merged pool
        idx = secrets.randbelow(len(candidate))
        candidate[idx] = secrets.choice(merged)
        secure_shuffle(candidate)
    return "".join(candidate)  # fallback (very unlikely to still have triples)

def main():
    print("=== Imran's Pro Password Generator ===")
    length = read_int("Desired password length (8–64): ", 8, 64)
    how_many = read_int("How many passwords to generate (1–20): ", 1, 20)

    use_lower  = yes_no("Include lowercase letters?", True)
    use_upper  = yes_no("Include UPPERCASE letters?", True)
    use_digits = yes_no("Include digits?", True)
    use_symbols= yes_no("Include symbols?", True)
    avoid_confusing = yes_no("Avoid confusing characters (O/0, l/1, |, etc.)?", True)

    pools, merged = build_pool(use_lower, use_upper, use_digits, use_symbols, avoid_confusing)

    # Validate selections
    active_pools = [p for p in pools if p]
    if not active_pools:
        print("No character sets available. Please enable at least one type.")
        return
    if length < len(active_pools):
        print(f"Length too small. You selected {len(active_pools)} character types, "
              f"so length must be at least {len(active_pools)}.")
        return

    print("\nGenerating...\n")
    for i in range(1, how_many + 1):
        pw = generate_one(length, active_pools, merged)
        if not pw:
            print(f"{i}. (failed to generate with current settings; try a longer length)")
        else:
            print(f"{i}. {pw}")

if __name__ == "__main__":
    main()
