- import requests

def compress_gif(api_key, input_file_path, output_file_path):
    url = 'https://api.tinify.com/shrink'
    auth = ('api', api_key)

    with open(input_file_path, 'rb') as file:
        response = requests.post(url, auth=auth, files={'input': file})

    if response.status_code == 201:
        result_url = response.headers['Location']
        response = requests.get(result_url, auth=auth, stream=True)

        with open(output_file_path, 'wb') as output_file:
            for chunk in response.iter_content(chunk_size=1024):
                output_file.write(chunk)

        print('Compression successful!')
    else:
        print('Compression failed:', response.json())

# Usage example
api_key = 'YOUR_API_KEY'
input_file_path = 'input.gif'
output_file_path = 'output.gif'

compress_gif(api_key, input_file_path, output_file_path)
