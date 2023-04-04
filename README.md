## Vulnerability 1: SQL Injection
### Discovery
All input fields rely on client-side input sanitization to prevent characters such as `'`, `=`, etc. from appearing. Since client-side sanitization is easily disabled in the browser using the inspector tool (done in my video by simply having the `onSubmit` action of the form to be to `return true` instead of calling whatever validation function was being used), all input forms on the website are vulnerable to SQL injection.
### Impact
After disabling the client-side sanitization, the most simple way to exploit this vulnerability is to skip password verification while logging in. This can be done generally by simply using the string
```SQL
" OR 1=1--
```
as the username, which simply gives access to the first account in the `users` table, `lhoneycutt`, which is an account with elevated privileges (a staff account). Alternatively, a specific account can be accessed by using the format:
```SQL
<username>" AND 1=1--
```
where `<username>` represents the desired account to be accessed. As per the criteria for demonstration, the exact input and the method/location for submission have been documented.
### Mitigation
To mitigate this vulnerability, input sanitization should take place server-side and not client-side. In essence, this exploit is possible because the site trusts the user not to disable their input sanitization, since it is running client-side. Migrating this functionality to the server side should prevent this sort of vulnerability. Additionally, common best practices with SQL such as using prepared statements may also help mitigate this vulnerability.
## Vulnerability 2: Insecure Direct Object Reference (IDOR)
### Discovery
This vulnerability appears wherever there are directory urls present when a user-specific document is being given. Although most of these (the `/handouts` and `/images` directories, for example) don't have much effect, but knowledge of/access to the `/handins` directory gives users access to all student handins, not just their own. Additionally, some playing around with unsanitized inputs to the input forms (i.e. inputting `"hello"` as the username) results in error pages that reveal the existence of more unprotected directories, specifically the `/includes` directories, which contains the much more critical `db.php` and `user.php` files, as well as templates for all the site pages.
### Impact
In order to exploit the information about directories revealed in the website URLs, all one has to do is simply delete the filename part of their url. For example, if my handin for assignment 1 is at url `http://localhost:8888/handins/0db2296d4116d45321aca7196927d3c34c9a78b7.pdf`, I can access the entire `/handins` directory by deleting the pdf part of the url and accessing `http://localhost:8888/handins`. This brings up a filesystem view of all the files within the `/handins` directory, which are all the student handins. A student exploiting this vulnerability could easily reference all of their peer's handouts in an unauthorized manner. As per the criteria for demonstration, the URL change and the informatoin discovered has been documented.
### Mitigation
Mitigating this vulnerability would probably involve implementing proper access controls to certain directories - the entire filetree of the website is actually accessible without ever logging in if one knows the proper directory names! Although access controls on `.php`-served pages have been implemented, static pages have no such protections. Alternatively, the assets/files could also be retrieved dynamically so that only users with the proper authentication can ask for and receive a given file.
## Vulnerability 3: Cross-Site Scripting (XSS)
### Discovery
Disabling sanitization on input fields allows for the uploading and execution of arbitrary javascript on the website on anyone's profile using the comment forms especially, but also the edit profile form. Although there is server-side sanitization to remove all instances of the word "script", this can be bypassed
### Impact
While logged in as a staff member and after diabling the client-side input sanitization in the comment form input box, I can leave a comment on a student's page that looks like a regrade notification:
```HTML
Your regrade for Weather Node Repair Workshop has been posted (<a onClick=alert(3)>VIEW</a>)
```
The `onClick` field of the link allows for the inclusion of arbitrary javascript that will be executed when the user clicks on the link. Although the example used just raises a simple `alert()`, this could easily be extended further to steal the user's session cookies, make requests, or redirect the user to a different website. As per the criteria for demonstration, the exact form input and where to input it has been documented.
### Mitigation
In order to mitigate this type of attack, as mentioned before input sanitization should be moved to the server side and not done in a home-grown manner. This will help ensure the prevention of bad/malicious inputs making their way into the database itself without having to trust the user not to tamper with those safeguards.
## Vulnerability 4: File Upload
### Discovery
This vulnerability appears in the assignment resubmission (for students) and assignment/solution upload (for staff) pages - basically anywhere that a file is allowed to be uploaded! As all of these files have good reason to be accessed by other people (i.e. students looking at handouts, staff looking at handins for grading), uploading malicious files to these file drops presents an opportunity for malicious code execution.
### Impact
Although the website will throw an error if you try to upload a non `.pdf` file, this safeguard is relatively easily bypassed by simply adding `.pdf` before the actual extension of the filename. For example, the included `index.pdf.html` file could easily be uploaded despite not actually being a `.pdf` file, and would allow for an arbitrary page to be served to whoever tried to access that particular submission. In the case of my example file, all that is done is to execute arbitrary javascript code, but as previously discussed this could lead to all sorts of mischief such as stealing user cookies. However, this ability could also be used to simply host malicious files (flie types that the browser couldn't render were immediately downloaded when accessed) or to upload other `.html` or `.php` files with other functionality. For example the wiki describes a web server that could be uploaded and used in this way. As per the wiki, a file upload which violates a normal content restriction has been verified and I have also demonstrated how code can be executed from the uploaded file.
### Mitigation
Mitigating this vulnerability would probably involve sanitizing filetypes or filenames more carefully. Clearly, whatever algorithm that is currently being used to detect incorrect file types is currently insufficient. Making this more stringent (even to the point of perhaps rejecting filenames containing suspicious substrings such as `html`) would reduce the possibility that an attacker could get a malicious file onto the filesystem. Similarly, having the server do a heuristic scan over the content of each new file to see if it seems to have malicious elements (i.e. javascript code, etc.) could be another way to help prevent malicious flie from being inadvertently hosted on the server.