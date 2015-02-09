

# c4l-social-ipynb

An iPython Notebook for analyzing social feeds, specifically Code4Lib Conference social activity.
Clone or download this notebook and open with IP[y]: Notebook http://ipython.org.

This is what the notebook will look like when you open it:

## You'll need to install these modules if you want to use them.
e.g. From the command line: pip install matplotlib

    import matplotlib as plt
    import networkx as nx
    plt.rc('figure', figsize=(8, 5))

## Twitter
Borrowing from the example at http://nbviewer.ipython.org/gist/ellisonbg/3837783. 
### Twitter API now requires OAuth2 for searching.
My example below updates ellisonbg's example to accomodate Twitter's OAuth2 requirement.
Twython makes it easy. You just need to get an APP_KEY and APP_SECRET from Twitter.
See https://twython.readthedocs.org/en/latest/usage/starting_out.html

# Your secrets!
    APP_KEY = ''
    APP_SECRET = ''

    from twython import Twython
    twitter = Twython(APP_KEY, APP_SECRET, oauth_version=2)
    ACCESS_TOKEN = twitter.obtain_access_token()
    twitter = Twython(APP_KEY, access_token=ACCESS_TOKEN)

Search Twitter returning pages of 20 tweets.
max_id is used to to set an upper bound on the time of the most
recent tweet returned. The first query is run without max_id, then
the last (earliest) tweet from the result set is used as the max_id value for the next
set.

    query = "%23c4l15"
    n_pages = 100
    results = []
    retweets = []
    urls = []
    max_id_next_page = 0
    # First query without max_id
    search = twitter.search(q=query, lang='en', count=20)
    for page in range(1, n_pages+1):
        res = search['statuses']
        # With max_id set, the first result will
        # be a repeat of the last result from the previous query.
        # Delete it.
        if max_id_next_page != 0:
          res = res[1:]
        
        if not res:
            print 'Stopping at page:', page
            break
            
        for t in res:
            if t['text'].startswith('RT '):
                retweets.append(t)
            else:
                results.append(t)
            for u in t['entities']['urls']:
                urls.append(u['expanded_url'])
        earliest_result = res[len(res)-1]
        max_id_next_page = earliest_result['id']
        search = twitter.search(q=query, lang='en', count=20, max_id=max_id_next_page)
        
    tweets = [t['text'] for t in results]
    urls_uniq = set(urls)
    # Quick summary
    print 'query: ', query
    print 'results: ', len(results)
    print 'retweets: ', len(retweets)
    print 'unique urls: ', len(urls_uniq)
    print 'urls mentioned: ', urls_uniq
    print 'tweets: ', tweets
    
    import nltk
    
    # nltk.download()
    
    lines = ([tweet['text'].encode('utf-8') for tweet in results])
    lines_joined = '\n'.join(lines)

NLTK includes various tokenizers, but they split punctuation from words. Not so helpful with URLs, hashtags, and user handles:

    #from nltk.tokenize import TreebankWordTokenizer
    #tokenizer = TreebankWordTokenizer()
    #tokens = tokenizer.tokenize(lines_joined)
    #print tokens
    
    #from nltk.tokenize import PunktWordTokenizer
    #tokenizer = PunktWordTokenizer()
    #tokenizer.tokenize(lines_joined)
    #print tokens
    
    #from nltk.tokenize import WordPunctTokenizer
    #word_punct_tokenizer = WordPunctTokenizer()
    #word_punct_tokenizer.tokenize(lines_joined)
    #print tokens

Just split on whitespace instead

    words = lines_joined.split()
    words = words
    print len(words)
    
    stop_words_url = 'http://www.textfixer.com/resources/common-english-words.txt'
    import urllib2
    response = urllib2.urlopen(stop_words_url)
    html = response.read()
    stop_words = html.split(',')
    print 'Stop words count:', len(stop_words)
    def stop_words_filter(words, stop_word):
        try:
            words = [words.remove(stop_word) for w in words]
        except ValueError:
            pass
        return words
    # Recursively remove stop_words
    words_filtered = [stop_words_filter(words, sw) for sw in stop_words]
    # Flatten
    words_filtered = [val for sublist in words_filtered for val in sublist]
    print 'Words with stop words removed:', len(words_filtered)
    
    print words_filtered
