<template>
  <div class="admin-gallery">
    <header class="admin-header">
      <h1>Gallery CMS</h1>
      <div class="header-actions">
        <a class="back-link" href="#home">Back to Site</a>
        <button class="btn logout" type="button" @click="logout">Logout</button>
      </div>
    </header>

    <section class="panel">
      <h2>{{ form.imageID ? 'Edit Image' : 'Add New Image' }}</h2>
      <form @submit.prevent="handleSubmit">
        <div class="form-row">
          <label>Project</label>
          <select v-model="projectMode">
            <option value="existing">Select existing</option>
            <option value="new">Create new</option>
          </select>
        </div>
        <div v-if="projectMode === 'existing'" class="form-row">
          <label>Choose Project</label>
          <select v-model="selectedProjectId">
            <option value="">Unassigned</option>
            <option v-for="project in projects" :key="project.projectID" :value="String(project.projectID)">
              {{ project.project_name }} (ID: {{ project.projectID }})
            </option>
          </select>
        </div>
        <div v-else class="form-row">
          <label>New Project Name</label>
          <input type="text" v-model.trim="newProjectName" placeholder="Project name" required />
        </div>
        <div v-if="projectMode === 'new'" class="form-row">
          <label>Project Description</label>
          <textarea
            v-model.trim="newProjectDescription"
            placeholder="Short project description"
            rows="3"
          ></textarea>
        </div>
        <div class="form-row">
          <label>Image File</label>
          <input type="file" accept="image/*" @change="handleFileChange" :required="!form.imageID" />
        </div>
        <div v-if="previewUrl" class="preview">
          <img :src="previewUrl" alt="Preview" />
        </div>
        <div class="form-actions">
          <button type="submit" class="btn primary">{{ form.imageID ? 'Update' : 'Create' }}</button>
          <button type="button" class="btn" @click="resetForm">Clear</button>
        </div>
      </form>
    </section>

    <section class="panel">
      <h2>Gallery Images</h2>
      <div v-if="loading" class="status">Loading...</div>
      <div v-else-if="images.length === 0" class="status">No images yet.</div>
      <div v-else class="grid">
        <div
          class="card"
          :class="{ draggable: !reordering, 'drag-over': dragOverIndex === index }"
          v-for="(image, index) in images"
          :key="image.imageID"
          :draggable="!reordering"
          @dragstart="onDragStart(index, $event)"
          @dragover.prevent="onDragOver(index)"
          @drop.prevent="onDrop(index)"
          @dragend="onDragEnd"
        >
          <div class="thumb">
            <img :src="imageUrl(image.imageID)" :alt="`Project ${image.projectID}`" />
          </div>
          <div class="meta">
            <div><strong>ID:</strong> {{ image.imageID }}</div>
            <div><strong>Project:</strong> {{ projectLabel(image.projectID) }}</div>
            <div class="actions">
              <button class="btn" :disabled="reordering || isFirst(index)" @click="moveImage(index, -1)">Up</button>
              <button class="btn" :disabled="reordering || isLast(index)" @click="moveImage(index, 1)">Down</button>
              <button class="btn" @click="editImage(image)">Edit</button>
              <button class="btn danger" @click="deleteImage(image.imageID)">Delete</button>
            </div>
          </div>
        </div>
      </div>
    </section>
  </div>
</template>

<script>
const API_BASE = process.env.VUE_APP_API_BASE || 'http://localhost:3000';

export default {
  name: 'AdminGallery',
  data() {
    return {
      images: [],
      projects: [],
      loading: false,
      projectMode: 'existing',
      selectedProjectId: '',
      newProjectName: '',
      newProjectDescription: '',
      form: {
        imageID: null,
        imageFile: null
      },
      previewUrl: '',
      reordering: false,
      draggedIndex: null,
      dragOverIndex: null
    }
  },
  mounted() {
    this.fetchImages();
    this.fetchProjects();
  },
  methods: {
    logout() {
      this.$emit('logout');
    },
    async authorizedFetch(url, options = {}) {
      const token = localStorage.getItem('admin_token');
      const headers = options.headers ? { ...options.headers } : {};
      if (token) {
        headers.Authorization = `Bearer ${token}`;
      }
      const response = await fetch(url, { ...options, headers });
      if (response.status === 401) {
        this.logout();
        throw new Error('Unauthorized');
      }
      return response;
    },
    async fetchProjects() {
      try {
        const response = await fetch(`${API_BASE}/project`);
        this.projects = await response.json();
      } catch (error) {
        console.error('Failed to load projects', error);
      }
    },
    async createProjectIfNeeded() {
      if (this.projectMode !== 'new') {
        return null;
      }
      const name = this.newProjectName.trim();
      if (!name) {
        throw new Error('Project name is required');
      }
      const response = await this.authorizedFetch(`${API_BASE}/project`, {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          project_name: name,
          project_description: this.newProjectDescription.trim() || null
        })
      });
      const created = await response.json();
      this.projects = [created, ...this.projects];
      return String(created.projectID);
    },
    async fetchImages() {
      this.loading = true;
      try {
        const response = await fetch(`${API_BASE}/project-image`);
        this.images = await response.json();
      } catch (error) {
        console.error('Failed to load images', error);
        alert('Failed to load images.');
      } finally {
        this.loading = false;
      }
    },
    async handleSubmit() {
      try {
        const createdProjectId = await this.createProjectIfNeeded();
        const projectIdToUse = this.projectMode === 'new'
          ? createdProjectId
          : (this.selectedProjectId || null);

        const formData = new FormData();
        if (projectIdToUse) {
          formData.append('projectID', projectIdToUse);
        }
        if (this.form.imageFile) {
          formData.append('image', this.form.imageFile);
        }
        if (this.form.imageID) {
          await this.authorizedFetch(`${API_BASE}/project-image/${this.form.imageID}`, {
            method: 'PATCH',
            body: formData
          });
        } else {
          await this.authorizedFetch(`${API_BASE}/project-image`, {
            method: 'POST',
            body: formData
          });
        }
        this.resetForm();
        await this.fetchImages();
      } catch (error) {
        console.error('Save failed', error);
        alert('Save failed.');
      }
    },
    editImage(image) {
      this.form = {
        imageID: image.imageID,
        imageFile: null
      };
      this.projectMode = 'existing';
      this.selectedProjectId = image.projectID ? String(image.projectID) : '';
      this.newProjectName = '';
      this.previewUrl = this.imageUrl(image.imageID);
      window.scrollTo({ top: 0, behavior: 'smooth' });
    },
    async deleteImage(id) {
      if (!confirm('Delete this image?')) {
        return;
      }
      try {
        await this.authorizedFetch(`${API_BASE}/project-image/${id}`, { method: 'DELETE' });
        await this.fetchImages();
      } catch (error) {
        console.error('Delete failed', error);
        alert('Delete failed.');
      }
    },
    isFirst(index) {
      return index === 0;
    },
    isLast(index) {
      return index === this.images.length - 1;
    },
    async moveImage(index, direction) {
      const targetIndex = index + direction;
      if (targetIndex < 0 || targetIndex >= this.images.length || this.reordering) {
        return;
      }

      this.reorderLocal(index, targetIndex);

      try {
        this.reordering = true;
        await this.persistOrder();
      } catch (error) {
        console.error('Reorder failed', error);
        alert('Reorder failed.');
        await this.fetchImages();
      } finally {
        this.reordering = false;
      }
    },
    async persistOrder() {
      await this.authorizedFetch(`${API_BASE}/project-image/reorder`, {
        method: 'PATCH',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ imageIds: this.images.map((image) => image.imageID) })
      });
    },
    reorderLocal(fromIndex, toIndex) {
      const updated = [...this.images];
      const [moved] = updated.splice(fromIndex, 1);
      updated.splice(toIndex, 0, moved);
      this.images = updated;
    },
    onDragStart(index, event) {
      if (this.reordering) {
        return;
      }
      this.draggedIndex = index;
      this.dragOverIndex = index;
      if (event?.dataTransfer) {
        event.dataTransfer.effectAllowed = 'move';
        event.dataTransfer.setData('text/plain', String(index));
      }
    },
    onDragOver(index) {
      if (this.draggedIndex === null || this.reordering) {
        return;
      }
      this.dragOverIndex = index;
    },
    async onDrop(index) {
      if (this.draggedIndex === null || this.reordering) {
        this.onDragEnd();
        return;
      }

      const fromIndex = this.draggedIndex;
      const toIndex = index;
      this.onDragEnd();

      if (fromIndex === toIndex) {
        return;
      }

      this.reorderLocal(fromIndex, toIndex);

      try {
        this.reordering = true;
        await this.persistOrder();
      } catch (error) {
        console.error('Reorder failed', error);
        alert('Reorder failed.');
        await this.fetchImages();
      } finally {
        this.reordering = false;
      }
    },
    onDragEnd() {
      this.draggedIndex = null;
      this.dragOverIndex = null;
    },
    resetForm() {
      this.form = { imageID: null, imageFile: null };
      this.projectMode = 'existing';
      this.selectedProjectId = '';
      this.newProjectName = '';
      this.newProjectDescription = '';
      this.previewUrl = '';
    },
    handleFileChange(event) {
      const file = event.target.files && event.target.files[0];
      this.form.imageFile = file || null;
      this.previewUrl = file ? URL.createObjectURL(file) : '';
    },
    imageUrl(id) {
      return `${API_BASE}/project-image/${id}/file`;
    },
    projectLabel(projectId) {
      if (!projectId) {
        return 'Unassigned';
      }
      const project = this.projects.find((item) => item.projectID === projectId);
      if (!project) {
        return `ID ${projectId}`;
      }
      return `${project.project_name} (ID: ${project.projectID})`;
    }
  }
}
</script>

<style scoped>
.admin-gallery {
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
  padding: 32px;
  color: #1f2937;
  background: #f8fafc;
  min-height: 100vh;
}

.admin-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 24px;
}

.header-actions {
  display: flex;
  align-items: center;
  gap: 12px;
}

.admin-header h1 {
  margin: 0;
  color: #111827;
}

.back-link {
  text-decoration: none;
  color: #2563eb;
  font-weight: 600;
}

.panel {
  background: #fff;
  border-radius: 12px;
  padding: 20px;
  box-shadow: 0 8px 20px rgba(15, 23, 42, 0.08);
  margin-bottom: 24px;
}

.form-row {
  display: flex;
  flex-direction: column;
  margin-bottom: 12px;
}

.form-row label {
  font-weight: 600;
  margin-bottom: 6px;
}

.form-row input {
  padding: 10px 12px;
  border: 1px solid #d1d5db;
  border-radius: 8px;
  font-size: 14px;
}

.form-row textarea {
  padding: 10px 12px;
  border: 1px solid #d1d5db;
  border-radius: 8px;
  font-size: 14px;
  resize: vertical;
  font-family: inherit;
}

.form-row select {
  padding: 10px 12px;
  border: 1px solid #d1d5db;
  border-radius: 8px;
  font-size: 14px;
  background: #fff;
}

.form-actions {
  display: flex;
  gap: 12px;
}

.preview {
  margin-bottom: 16px;
}

.preview img {
  width: 100%;
  max-height: 240px;
  object-fit: cover;
  border-radius: 10px;
}

.btn {
  padding: 10px 16px;
  border: none;
  border-radius: 8px;
  cursor: pointer;
  background: #e5e7eb;
  color: #111827;
  font-weight: 600;
}

.btn.primary {
  background: #2563eb;
  color: #fff;
}

.btn.logout {
  background: #0f172a;
  color: #fff;
}

.btn.danger {
  background: #ef4444;
  color: #fff;
}

.status {
  color: #6b7280;
}

.grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(220px, 1fr));
  gap: 16px;
}

.card {
  border: 1px solid #e5e7eb;
  border-radius: 12px;
  overflow: hidden;
  background: #fff;
}

.card.draggable {
  cursor: grab;
}

.card.drag-over {
  border-color: #2563eb;
  box-shadow: 0 0 0 2px rgba(37, 99, 235, 0.2);
}

.thumb {
  height: 160px;
  background: #f3f4f6;
  display: flex;
  align-items: center;
  justify-content: center;
}

.thumb img {
  width: 100%;
  height: 100%;
  object-fit: cover;
}

.meta {
  padding: 12px;
}

.actions {
  display: flex;
  gap: 8px;
  margin-top: 12px;
}
</style>
