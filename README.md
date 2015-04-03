#  
The Spamassassin Milter Plugin can be tricked into executing any command
as the root user remotely.
If spamass-milter is run with the expand flag (-x option) it runs a
popen() including the attacker supplied 
recipient (RCPT TO).
 
>From spamass-milter-0.3.1 (-latest) Line 820:
 
//
// Gets called once for each recipient
//
// stores the first recipient in the spamassassin object and
// stores all addresses and the number thereof (some redundancy)
//
 
sfsistat
mlfi_envrcpt(SMFICTX* ctx, char** envrcpt)
{
        struct context *sctx = (struct context*)smfi_getpriv(ctx);
        SpamAssassin* assassin = sctx->assassin;
        FILE *p;
#if defined(__FreeBSD__)
        int rv;
#endif
 
        debug(D_FUNC, "mlfi_envrcpt: enter");
 
        if (flag_expand)
        {
                /* open a pipe to sendmail so we can do address
expansion */
 
                char buf[1024];
                char *fmt="%s -bv \"%s\" 2>&1";
 
#if defined(HAVE_SNPRINTF)
                snprintf(buf, sizeof(buf)-1, fmt, SENDMAIL, envrcpt[0]);
#else
                /* XXX possible buffer overflow here // is this a
joke ?! */
                sprintf(buf, fmt, SENDMAIL, envrcpt[0]);
#endif
 
                debug(D_RCPT, "calling %s", buf);
 
#if defined(__FreeBSD__) /* popen bug - see PR bin/50770 */
                rv = pthread_mutex_lock(&popen_mutex);
                if (rv)
                {
                        debug(D_ALWAYS, "Could not lock popen mutex: %
s", strerror(rv));
                        abort();
                }
#endif
 
                p = popen(buf, "r");                [1]
                if (!p)
                {
                        debug(D_RCPT, "popen failed(%s).  Will not
expand aliases", strerror(errno));
                        assassin->expandedrcpt.push_back(envrcpt[0]);
 
 
[1] the vulnerable popen() call.
