import os
import json
from flask import Flask, render_template, request, redirect, url_for

app = Flask(__name__)
NOTES_FILE = 'notes.json'

# 加载笔记数据
def load_notes():
    if os.path.exists(NOTES_FILE):
        with open(NOTES_FILE, 'r', encoding='utf-8') as file:
            return json.load(file)
    return []

# 保存笔记数据
def save_notes(notes):
    with open(NOTES_FILE, 'w', encoding='utf-8') as file:
        json.dump(notes, file, ensure_ascii=False, indent=4)

# 首页，显示所有笔记
@app.route('/')
def index():
    notes = load_notes()
    return render_template('index.html', notes=notes)

# 创建新笔记
@app.route('/create', methods=['GET', 'POST'])
def create():
    if request.method == 'POST':
        title = request.form.get('title')
        content = request.form.get('content')
        notes = load_notes()
        new_note = {
            'id': len(notes) + 1,
            'title': title,
            'content': content
        }
        notes.append(new_note)
        save_notes(notes)
        return redirect(url_for('index'))
    return render_template('create.html')

# 查看单个笔记
@app.route('/note/<int:note_id>')
def note(note_id):
    notes = load_notes()
    for note in notes:
        if note['id'] == note_id:
            return render_template('note.html', note=note)
    return "笔记未找到", 404

# 编辑笔记
@app.route('/edit/<int:note_id>', methods=['GET', 'POST'])
def edit(note_id):
    notes = load_notes()
    for note in notes:
        if note['id'] == note_id:
            if request.method == 'POST':
                note['title'] = request.form.get('title')
                note['content'] = request.form.get('content')
                save_notes(notes)
                return redirect(url_for('index'))
            return render_template('edit.html', note=note)
    return "笔记未找到", 404

# 删除笔记
@app.route('/delete/<int:note_id>')
def delete(note_id):
    notes = load_notes()
    new_notes = [note for note in notes if note['id'] != note_id]
    save_notes(new_notes)
    return redirect(url_for('index'))

if __name__ == '__main__':
    app.run(debug=True)
