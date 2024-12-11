---
title: AI - OSINT - Solving GitHub Alias
tags: ai osint digital-identity github ethical-hacking cybersecurity recon social-engineering reconnaissance cyber-investigation opensource-intelligence
---

# Title

How AI-Powered OSINT Helped Me Solve the Mystery of My GitHub Alias and Reclaim My Digital Identity !

# Description

A few years ago, while setting up my GitHub account and trying to establish my digital presence, I discovered that the username `AB2` was already taken. The account belonged to someone else, which was surprising since I had envisioned that alias as part of my online identity.

![Pasted image 20230222010512](https://github.com/user-attachments/assets/990ac5dc-1ac2-43d2-9ecc-37f6bddd688f)

Fast forward to two days ago, I decided it was time to revisit this. I wanted to find out who owned the ab2 account and explore the possibility of contacting them to request the transfer of this digital identity.

# Recon

First, I visited the ab2 profile page to understand what kind of activity or projects the user had been involved in.

I scrolled through their repositories and spotted one that had been committed to most recently. It was clear that this user was active, employing GitHub Actions as part of their workflows. This was important because it meant the account was still in use and not abandoned.

![Pasted image 20230222010540](https://github.com/user-attachments/assets/ca7af191-1907-49da-b916-b1b101162767)

Since this repository was not a fork but rather an original project, I decided to take the next step: cloning it locally to further analyze its history.

# Digging Deeper

I cloned the repository:

```bash
git clone https://github.com/ab2/tech-radar.git
```

Once cloned, I used the command `git log --reverse` to sift through the repository's commit history, looking for the first or initial commit.

```bash
git log --reverse
```

In the output, I found an essential detail: the initial commit included the author's email address. This provided me with a way to reach out directly.

![Pasted image 20230222010711](https://github.com/user-attachments/assets/b26b602b-ac7b-40cd-b5d2-c822ab8764ce)


# Crafting the Email

Now that I had the contact details, I needed to draft an email that would be both respectful and persuasive. This is where AI came in handy. I turned to ChatGPT to help me compose a concise, friendly message that would capture my intention.

![Pasted image 20230222010759](https://github.com/user-attachments/assets/93d1692f-6cb7-489d-9496-3fb43268c33f)

# The Response

I sent out the email and waited. To my surprise and relief, I received a reply the following day. The user, who turned out to be very open and understanding, welcomed the idea and even asked for my help in guiding them through the steps needed to transfer the account.

![Pasted image 20230222010832](https://github.com/user-attachments/assets/3c4e82fd-200f-45fa-9df4-254b1674a52f)

# Conclusion

This experience highlighted not only the power of OSINT and traditional research methods but also the new ways AI can assist in digital investigations. From finding hidden email addresses to drafting outreach emails, technology played a pivotal role in reconnecting with my digital identity. The outcome was a success, proving that sometimes, with a little persistence and the right tools, even the most challenging puzzles can be solved.

![image](https://github.com/user-attachments/assets/0203be86-1040-4fe1-b368-42b6cb2b49ba)
