```python
import argparse
import ciso8601
import configparser
import pandas as pd
import privatebinapi as pb
from elasticsearch import Elasticsearch, helpers
```

```python
class ChatMessage:
    def __init__(self, source):
        self.timestamp = ciso8601.parse_datetime(source['@timestamp'])
        self.username = source['username']
        self.message = source['message']

    def _asdict(self):
        def get_date(timestamp):
            return timestamp.strftime('%Y-%m-%d')

        def get_time(timestamp):
            return timestamp.strftime('%H:%M')

        message_dict = {
            'date': get_date(self.timestamp),
            'time': get_time(self.timestamp),
            'username': self.username,
            'message': self.message
        }
        return message_dict
```

```python
def get_elasticsearch_data(username):
    es = Elasticsearch([config_section_map('default')['elastic']])
    results = helpers.scan(
        es,
        index=config_section_map('default')['index'],
        preserve_order=True,
        query={
            'query': {
                'bool': {
                    'must': [],
                    'filter': [{'match_phrase': {'username': username}}]
                }
            }
        }
    )
    return results
```

```python
def parse_messages(results):
    messages = []
    for result in results:
        source = result['_source']
        message = ChatMessage(source)
        messages.append(message._asdict())
    return messages
```

```python
def get_message_df(messages):
    return pd.DataFrame.from_dict(messages)
```

```python
def markdown_messages(messages):
    return messages.to_markdown()
```

```python
def messages_to_csv(messages, outfile):
    return messages.to_csv(outfile, index=False, encoding='utf-8')
```

```python
def messages_to_paste(messages, expiration_time):
    submission = pb.send(
        config_section_map('default')['site'],
        formatting='markdown',
        text=messages,
        expiration=expiration_time
    )
    return submission['full_url']
```

```python
def parse_args():
    parser = argparse.ArgumentParser()
    parser.add_argument('-u', '--username', required=True)
    parser.add_argument('-e', '--expiration', default='1week', required=False)
    args = parser.parse_args()
    return args
```

```python
def config_section_map(section):
    options_dict = {}
    Config = configparser.ConfigParser()
    Config.read('./conf/settings.ini')
    options = Config.options(section)
    for option in options:
        options_dict[option] = Config.get(section, option)
    return options_dict
```

```python
def main():
    messages = get_message_df(
        parse_messages(get_elasticsearch_data(arg.username))
    )
    md_messages = markdown_messages(messages)
    print(messages_to_paste(md_messages, arg.expiration))
```

```python
if __name__ == '__main__':
    if arg := parse_args():
        main()
```
