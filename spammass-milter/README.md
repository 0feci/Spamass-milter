<h3>Spamass-milter local Root Command Execution</h3>

The Spamassassin Milter Plugin can be tricked into executing any command as the root user remotely.
If spamass-milter is run with the expand flag (-x option) it runs a
popen() including the attacker supplied 
recipient (RCPT TO).
<br></br>
For details vulnerability : http://www.exploit-db.com/exploits/11662/
