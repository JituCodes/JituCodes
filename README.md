import os
from flask import Flask, request, jsonify
from werkzeug.utils import secure_filename

app = Flask(__name__)
UPLOAD_FOLDER = 'uploads'
ALLOWED_EXTENSIONS = {'gif'}

app.config['UPLOAD_FOLDER'] = UPLOAD_FOLDER
app.config['MAX_CONTENT_LENGTH'] = 16 * 1024 * 1024  # 16MB maximum file size


def allowed_file(filename):
    return '.' in filename and \
           filename.rsplit('.', 1)[1].lower() in ALLOWED_EXTENSIONS


def compress_gif(input_path, output_path, quality):
    cmd = f'ffmpeg -i {input_path} -vf "fps=10,scale=320:-1:flags=lanczos" -c:v gif -b:v {quality} {output_path}'
    os.system(cmd)


@app.route('/compress', methods=['POST'])
def compress():
    # Check if the POST request has the file part
    if 'file' not in request.files:
        return jsonify({'error': 'No file part in the request'})

    file = request.files['file']

    if file.filename == '':
        return jsonify({'error': 'No file selected'})

    if file and allowed_file(file.filename):
        filename = secure_filename(file.filename)
        file_path = os.path.join(app.config['UPLOAD_FOLDER'], filename)
        file.save(file_path)

        # Compress the GIF
        output_path = os.path.join(app.config['UPLOAD_FOLDER'], 'compressed_' + filename)
        compress_gif(file_path, output_path, '1M')

        # Return the compressed GIF
        return jsonify({'compressed_file': output_path})

    return jsonify({'error': 'Invalid file format. Only GIF files are allowed.'})


if __name__ == '__main__':
    app.run()
