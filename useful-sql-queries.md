# Useful SQL Table info & Queries used for Wordpress Administration 

## Backup your WordPress Database using phpMyAdmin

## Before you proceed with any changes, be sure to backup your database. It is a good practice to always backup your database before making any major changes. This ensures that even if anything were to go wrong, you would still be able to restore it.

1. Login to your phpMyAdmin.
2. Select your WordPress database.
3. Click on Export at the top of the navigation.
4. Select the tables you want to backup, or select all tables to backup the whole database.
5. Select SQL to export as .sql extension.
6. Check the “Save as file” checkbox.
7. Choose compression type, select gzipped to compress the database to a smaller size.
8. Finally click Go, and a download window will prompt you to save your backup database file.

## Core Tables 
(wp_ prefix may differ depending on your installation)

wp_posts – all the content of your posts and pages as well as menu data and media attachments.<br>
wp_postmeta – meta data for each post. Meta data is added to this table when you add a custom field to your posts so for example, you could add what music you were listening to at the time of writing the post.<br>
wp_comments – all your comments on posts and pages including author, date, email, etc.<br>
wp_commentmeta – meta data for comments.<br>
wp_users – usernames, passwords (encrypted), and other user data.<br>
wp_usermeta – meta data for users.<br>
wp_options – general WordPress settings.<br>
wp_links – used for blogroll links, not really used on most WordPress sites today.<br>
wp_terms – categories and tags for posts.<br>
wp_termmeta – meta data for categories and tags.<br>
wp_term_relationships – links posts with categories and tags.<br>
wp_term_taxonomy – taxonomies are used for classifying your data. The WordPress default taxonomies are category, tag, and link category. This table manages the taxonomies including their name and description.<br>

## Core Table Fields

wp_users:<br>
ID<br>
user_login<br>
user_pass<br>
user_nicename<br>
user_email<br>
user_url<br>
user_registered<br>
user_activation_key<br>
user_status<br>
display_name<br>

# Useful SQL Queries 
(Login to phpMyAdmin panel and select your WordPress database.  Click on the SQL tab which will bring you to a page with a SQL query box.)

## Change SiteURL and HomeURL
```
UPDATE wp_options SET option_value = replace(option_value, 'http://www.oldsiteurl.com', 'http://www.newsiteurl.com') WHERE option_name = 'home' OR option_name = 'siteurl';
```
## Change GUID
(Fix the URLs for the GUID field in wp_posts table after migration) 
```
UPDATE wp_posts SET guid = REPLACE (guid, 'http://www.oldsiteurl.com', 'http://www.newsiteurl.com');
```
## Change URLs of post content items 
(Fix content paths after migration e.g. absolute image paths in Media Library)
```
UPDATE wp_posts SET post_content = REPLACE (post_content, 'http://www.oldsiteurl.com', 'http://www.newsiteurl.com');
```
## Change Image Path Only
(If using a CDN for example)
```
UPDATE wp_posts SET post_content = REPLACE (post_content, 'src="http://www.oldsiteurl.com', 'src="http://yourcdn.newsiteurl.com');
```
You will also need to update the GUID for image attachments
```
UPDATE wp_posts SET  guid = REPLACE (guid, 'http://www.oldsiteurl.com', 'http://yourcdn.newsiteurl.com') WHERE post_type = 'attachment';
```
## Update URLs in post meta data
```
UPDATE wp_postmeta SET meta_value = REPLACE (meta_value, 'http://www.oldsiteurl.com','http://www.newsiteurl.com');
```
## Change user password
```
UPDATE wp_users SET user_pass = MD5( '[new_password]' ) WHERE user_login = '[username]';
```
## Change Username
```
UPDATE wp_users SET user_login = 'newusername' WHERE user_login = 'oldusername';
```
## Search and Replace content text in post tables
```
UPDATE wp_posts SET post_content = REPLACE (post_content,'Item to replace here','Replacement text here');
```
## Assign Posts to a New Author
```
UPDATE wp_posts SET post_author = (SELECT ID FROM wp_users WHERE user_login = '[new_author_login]') WHERE post_author = (SELECT ID FROM wp_users WHERE user_login = '[old_author_login]');
```
## Bulk Delete Spam Comments
```
DELETE FROM wp_comments WHERE comment_approved = "spam";
```
## Delete all comments (including genuine ones)
```
DELETE FROM wp_comments WHERE comment_approved = "0";
```
## Delete comments attributed to a specific url (Muliple spammers posting the same url or single spammer posting the same url multiple times)
```
DELETE from wp_comments WHERE comment_author_url LIKE "%spamurl%" ;
```
## Delete all Pingback
```
DELETE FROM wp_comments WHERE comment_type = 'pingback';
```
## Change Posts into Pages
```
UPDATE wp_posts SET post_type = 'page' WHERE post_type = 'post';
```
## Change Pages into posts
```
UPDATE wp_posts SET post_type = 'post' WHERE post_type = 'page';
```
## Delete Post Revisions
```
DELETE a,b,c FROM wp_posts a LEFT JOIN wp_term_relationships b ON (a.ID = b.object_id) LEFT JOIN wp_postmeta c ON (a.ID = c.post_id) WHERE a.post_type = 'revision';
```
## Disable Comments on Old Posts
```
UPDATE wp_posts SET comment_status = 'closed' WHERE post_date &lt; '2016-01-01' AND post_status = 'publish';
```
## Change URL of WordPress Images
```
UPDATE wp_posts SET post_content = replace(post_content, 'Old URL', 'New URL');
```
## Batch Disable Plugins
```
UPDATE wp_options SET option_value = '' WHERE option_name = 'active_plugins';
```
## Disable Comments on All Posts
```
UPDATE wp_posts SET comment_status = 'closed' where post_type ='post';
```
## After removing a plugin sometimes Meta data is still stored in the post_meta table.  Remove this with 
```
DELETE FROM wp_postmeta WHERE meta_key = 'your-meta-key';
```
## Export comment author email addresses with no duplicates
(Once you have the result, under Query results operations, select export to export all the emails in phpMyAdmin.)
```
SELECT DISTINCT comment_author_email FROM wp_comments;
```
## Identify Unused Tags
(remove redundant tags in cloud/listing)
```
SELECT * From wp_terms wt
INNER JOIN wp_term_taxonomy wtt ON wt.term_id=wtt.term_id WHERE wtt.taxonomy='post_tag' AND wtt.count=0;
```


