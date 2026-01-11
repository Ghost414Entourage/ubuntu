#!/bin/bash

# Require a search term
if [ -z "$1" ]; then
    echo "Usage: $0 <search-term>"
    exit 1
fi

SEARCH_TERM="$1"

# Run apt search, filter only package result lines
mapfile -t results < <(apt search "$SEARCH_TERM" 2>/dev/null | grep -E '^[a-z0-9.+-]+/')

if [ ${#results[@]} -eq 0 ]; then
    echo "No results found for '$SEARCH_TERM'."
    exit 1
fi

# Show results with numbers and installed status
echo "Packages found for '$SEARCH_TERM':"
for i in "${!results[@]}"; do
    pkg=$(echo "${results[$i]}" | cut -d'/' -f1)
    if dpkg -s "$pkg" &>/dev/null; then
        status="[INSTALLED]"
    else
        status="[NOT INSTALLED]"
    fi
    desc=$(echo "${results[$i]}" | cut -d' ' -f2-)
    printf "%2d) %-30s %-13s : %s\n" $((i+1)) "$pkg" "$status" "$desc"
done

# Ask user for choices
read -p "Enter the number(s) of the package(s) you want to install (space-separated): " -a choices

if [ ${#choices[@]} -eq 0 ]; then
    echo "No choices entered."
    exit 0
fi

# Build package list
pkgs=()
for choice in "${choices[@]}"; do
    if ! [[ "$choice" =~ ^[0-9]+$ ]] || [ "$choice" -le 0 ] || [ "$choice" -gt ${#results[@]} ]; then
        echo "Skipping invalid choice: $choice"
        continue
    fi
    pkg=$(echo "${results[$((choice-1))]}" | cut -d'/' -f1)
    # Only add if not already installed
    if dpkg -s "$pkg" &>/dev/null; then
        echo "Already installed: $pkg"
    else
        pkgs+=("$pkg")
    fi
done

if [ ${#pkgs[@]} -eq 0 ]; then
    echo "No new packages to install."
    exit 0
fi

# Confirm and install
echo "Installing packages: ${pkgs[*]}"
sudo apt install -y "${pkgs[@]}"
