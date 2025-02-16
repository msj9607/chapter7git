# chapter7git
chapter7git
#!/bin/bash

# Function to print usage statement
usage() {
    echo "Usage: $0 -u <usernames> | -f <file> [--help]"
    echo "  -u  Accept multiple account usernames as arguments"
    echo "  -f  Accept a filename containing usernames (one per line)"
    echo "  --help  Show this help message and exit"
    exit 1
}

# Function to create a new user
create_user() {
    local username=$1
    local password=$(openssl rand -base64 12)
    
    if id -u "$username" >/dev/null 2>&1; then
        echo "User '$username' already exists. Skipping."
    else
        if [[ $username =~ ^[a-zA-Z0-9._-]+$ ]]; then
            sudo useradd -m -s /bin/bash "$username"
            echo "$username:$password" | sudo chpasswd
            sudo chage -d 0 "$username"
            echo "$username:$password" >> new_users.txt
            echo "User '$username' created successfully."
        else
            echo "Invalid username '$username'. Skipping."
        fi
    fi
}

# Check if no arguments are provided
if [ "$#" -eq 0 ]; then
    usage
fi

# Parse command-line arguments
while [ "$#" -gt 0 ]; do
    case "$1" in
        -u)
            shift
            usernames=$@
            ;;
        -f)
            shift
            file=$1
            ;;
        --help)
            usage
            ;;
        *)
            usage
            ;;
    esac
    shift
done

# Process usernames from command-line arguments
if [ -n "$usernames" ]; then
    for username in $usernames; do
        create_user "$username"
    done
fi

# Process usernames from file
if [ -n "$file" ]; then
    if [ -f "$file" ]; then
        while IFS= read -r username; do
            create_user "$username"
        done < "$file"
    else
        echo "File '$file' not found."
        exit 1
    fi
fi
