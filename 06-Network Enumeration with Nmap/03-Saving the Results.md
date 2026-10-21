# Nmap — Saving Scan Results and HTML Reporting

**HTB Academy | Network Enumeration with Nmap**

In this hands-on exercise, I practiced performing full TCP port scans with Nmap and saving the results in multiple formats for further analysis and documentation.

I explored three Nmap output formats: Normal (`.nmap`), Grepable (`.gnmap`), and XML (`.xml`). I also learned how to convert XML scan results into a structured HTML report using `xsltproc`.

## Commands Used

```bash
sudo nmap -sS -p- -oA target 10.129.141.101
xsltproc target.xml -o target.html
firefox target.html
```

## Key Skills

- Full TCP port scanning
- Saving Nmap results using `-oA`
- Understanding Nmap output formats
- Generating HTML security reports
- Identifying and analyzing open TCP ports

**Key Takeaway:** Saving and organizing scan results is essential for effective network enumeration, technical documentation, and professional penetration testing reporting.

<img width="878" height="372" alt="image" src="https://github.com/user-attachments/assets/d6a5b1d4-4c09-416e-b182-b689dc0346de" />

<img width="887" height="116" alt="image" src="https://github.com/user-attachments/assets/2a8a6dad-2b31-4303-81a4-717ee147cc57" />

<img width="1752" height="757" alt="image" src="https://github.com/user-attachments/assets/45d82a6e-702a-4ad5-8641-690a6bc3f97d" />

