<template>
<div>
<div class="row">
  <div class="col-3">
  <q-card class="table-bg no-shadow">
    <q-card-section>
      <div class="text-h6">Filtro por Categorías</div>
    </q-card-section>
    <q-card-section>
        <q-expansion-item
          popup 
          group="categoryFilter"
          expand-separator
          expand-icon-toggle
          v-for="(category, index) in categories"
          :key="category.id"
        >
          <q-item
            :label="subcategory.name"
            style="border-radius: 30px;"
            v-for="subcategory in categories[index].subcategories"
            :key="subcategory.id"
            :class="{selected: subcategory.selected}"
          >

            <q-item-section class="pointer q-ml-lg" @click="updateProductListWithSubcategory(subcategory); subcategory.selected = true">
              {{ subcategory.name }}
            </q-item-section>

          </q-item>

          <template v-slot:header>

            <q-item-section :class="{selected: category.selected, pointer: true}" @click="updateProductListWithCategory(category); category.selected = true">
              {{category.name}}
            </q-item-section>

          </template>
        </q-expansion-item>

        <div
      class="q-ma-md">
        <q-btn
          v-if="activeCategoryFilter"
          color="positive"
          class="text-capitalize"
          @click="resetCategoryFilter()"
        >
        Restaurar
        </q-btn>
    </div>
        
    </q-card-section>
  </q-card>
  </div>

  <div class="col-9">
  <q-card class="table-bg no-shadow">
    <q-card-section>
      <div class="text-h6">Productos</div>
    </q-card-section>
    <q-separator color="white" />
    <q-card-section class="q-pa-none">
      <q-table
        class="table-bg"
        :data="products"
        :columns="columns"
        :visible-columns="visibleColumns"
        :filter="filter"
        :pagination="pagination"
      >
        <template v-slot:top-left>
              <q-input
                borderless
                dense
                debounce="300"
                v-model="filter"
                placeholder="Buscar"
              >
            <template v-slot:append>
              <q-icon name="search" />
            </template>
          </q-input>
          
        </template>
        <template v-slot:top-right>
          <q-btn
            color="positive"
            class="float-right text-capitalize"
            icon="add"
            @click="addProductDialog = true"
          />
        </template>

        <template v-slot:body-cell-Price="props">
          <td class="text-left">${{props.row.price.toFixed(2)}}</td>
        </template>

        <template v-slot:body-cell-Action="props">
          <q-td :props="props" >
            <q-btn
              icon="share"
              size="sm"
              class="q-ml-sm"
              flat
              dense
              @click="relateProduct(props.row)"
            />
            <q-btn
              icon="edit"
              size="sm"
              class="q-ml-sm"
              flat
              dense
              @click="editProduct(props.row)"
            />
            <q-btn
              icon="delete"
              size="sm"
              class="q-ml-sm"
              flat
              dense
              @click="deleteProduct(props.row)"
            />
          </q-td>
        </template>
        <template v-slot:body-cell-Description="props">
          <q-td :props="props">
            {{props.row.description.substr(0, 10) + '&hellip;'}}
          </q-td>
        </template>
        <template v-slot:body-cell-Image="props">
          <q-td :props="props" style="height: 100px; width: 100px">
            <q-img
              v-if="props.row.image_url"
              :src="`http://api.pediapp.com.ar/images/` + props.row.image_url"
              fit="fill"
            />
          </q-td>
        </template>
        <template v-slot:body-cell-Active="props">
          <q-td :props="props">
            <q-checkbox
              disable
              v-model="props.row.active"
            />
          </q-td>
        </template>
        <template v-slot:body-cell-Stock="props">
          <q-td :props="props">
            <q-checkbox
              disable
              v-model="props.row.stock"
            />
          </q-td>
        </template>
      </q-table>
    </q-card-section>
    <q-dialog v-model="addProductDialog" persistent>
      <q-card style="min-width: 750px">
        <q-card-section>
          <div class="text-h6">{{ formTitle }}</div>
        </q-card-section>

        <q-card-section class="q-pt-none">
          <div class="row">
            <div class="col q-mx-sm">
              <q-input dense v-model="editedItem.name" label="Nombre" />
            </div>
            <div class="col q-mx-sm">
              <q-input dense v-model="editedItem.price" label="Precio" />
            </div>
          </div>
          <div class="row q-mt-md">
            <div class="col q-mx-sm">
              <q-input
                dense
                v-model="editedItem.description"
                clearable
                autogrow
                label="Descripción"
                type="textarea"
              />
            </div>
          </div>
          <div class="row q-mt-md justify-center">
            <div class="col-2 q-mx-sm">
              <q-checkbox
              v-model="editedItem.active"
              label="Activo?"
              left-label           
              />              
            </div>
            <div class="col-2 q-mx-sm">
              <q-checkbox
              v-model="editedItem.stock"
              label="Tiene Stock?"
              left-label           
              />              
            </div>

          </div>
          <div class="row q-mt-md justify-center">
            <div class="col-6 q-mx-sm">
              <q-file
                rounded
                outlined
                type="file"
                bottom-slots
                v-model="editedItem.file"
                label="Foto Principal"
                counter
                max-file-size="2097152"
                max-files="1"
                accept=".jpg, image/*"
                @rejected="onRejected"
              >
                <template v-slot:before>
                  <q-icon name="image" />
                </template>

                <template v-slot:append>
                  <q-icon
                    v-if="editedItem.image_url !== null"
                    name="close"
                    @click.stop="editedItem.image_url = null"
                    class="cursor-pointer"
                  />
                  <q-icon name="search" @click.stop />
                </template>

                <template v-slot:hint> Tamaño máximo 2MB - 950 x 950 px. </template>
              </q-file>
            </div>
          </div>
          <div class="row q-mt-md justify-center">
            Fotos adjuntas
          </div>
          <div class="row q-mt-md justify-center">
            <div 
              v-for="(image, index) in attachedProductImages"
              :key="image.id"
            >
              <q-chip 
              removable
              @remove="attachedProductImages.splice(index, 1)">
                <q-img
                :src="`http://api.pediapp.com.ar/images/` + image.image_url"
                height="auto"
                width="50px"
                :ratio="1"
                />
              </q-chip>
            </div>
          </div>
          <div class="row q-mt-md justify-center">
            <div class="col-6 q-mx-sm">
              <q-file
                rounded
                outlined
                type="file"
                bottom-slots
                v-model="editedItem.additional_images"
                label="Fotos adicionales"
                counter
                max-file-size="2097152"
                max-files="5"
                multiple
                accept=".jpg, image/*"
                @rejected="onRejected"
              >
                <template v-slot:before>
                  <q-icon name="image" />
                </template>

                <template v-slot:append>
                  <q-icon
                    v-if="editedItem.additional_images !== null"
                    name="close"
                    @click.stop="editedItem.additional_images = null"
                    class="cursor-pointer"
                  />
                  <q-icon name="search" @click.stop />
                </template>

                <template v-slot:hint> Tamaño máximo 2MB - 950 x 950 px. </template>
              </q-file>
            </div>
          </div>
        </q-card-section>

        <q-card-actions align="right" class="text-primary">
          <q-btn flat label="Cancelar" @click="closeNewProductDialog" />
          <q-btn flat :label="formTitle" @click="saveNewProduct" />
        </q-card-actions>
      </q-card>
    </q-dialog>

    <q-dialog v-model="relateProductDialog" persistent>
      <q-card style="min-width: 750px">

        <q-card-section>
          <div class="text">Seleccionar Categoria/s o Subcategoría/s a relacionar</div>
        </q-card-section>

        <q-card-section class="scroll">
        <q-expansion-item
          popup 
          group="categoryRelations"
          expand-separator
          expand-icon-toggle
          v-for="(category, index) in categories"
          :key="category.id"
          
        >
          <q-item
            :label="subcategory.name"
            v-for="subcategory in categories[index].subcategories"
            :key="subcategory.id"
          >

            <q-item-section class="pointer" @click="setSelectedSubcategories(category, subcategory)">
              {{ subcategory.name }}
            </q-item-section>

          </q-item>

          <template v-slot:header>

            <q-item-section  class="pointer" @click="setSelectedCategories(category)">
              {{category.name}}
            </q-item-section>

          </template>
        </q-expansion-item>

          <div>
            <div class="text">Categorías seleccionadas</div>
            <q-chip
            removable
            @remove="unsetSelectedCategories(selectedCategory, index)"
            v-for="(selectedCategory, index) in selectedCategories"
            :key="selectedCategory.id"
            :label="selectedCategory.name"
            icon="bookmark" color="blue" text-color="white"
            >
            </q-chip>
          </div>

          <div>
            <div class="text">Subcategorías seleccionadas</div>
            <q-chip
            removable
            @remove="selectedSubcategories.splice(index,1)"
            v-for="(selectedSubcategory, index) in selectedSubcategories"
            :key="selectedSubcategory.id"
            :label="selectedSubcategory.name"
            icon="bookmark" color="red" text-color="white"
            >
            
            </q-chip>
          </div>
        </q-card-section>

        <q-card-actions align="right" class="text-primary">
          <q-btn flat label="Cancelar" @click="closeRelationDialog" />
          <q-btn flat label="Relacionar Subcategorias" @click="saveNewSubcategoryRelation" />
        </q-card-actions>
      </q-card>
    </q-dialog>
  </q-card>
  </div>
</div>
<div class="row q-mt-lg">
  <div class="col-10">
  <q-card flat  >
      <q-card-section>
        <q-card-section>
      <div class="text-h6">Slider</div>
    </q-card-section>

        <q-card-section class="flex flex-center">
          <q-img
            class="rounded-borders"
            :src="`http://api.pediapp.com.ar/images/`+ productSection[0].slider_url"
          />
        </q-card-section>
      </q-card-section>
    </q-card>
    </div>
    <div class="column justify-center">
    <div class="col-5">
              <q-file
                rounded
                outlined
                type="file"
                bottom-slots
                v-model="productSlider"
                max-file-size="2097152"
                max-files="1"
                accept=".jpg, image/*"
                @rejected="onRejected"
              >
                <template v-slot:before>
                  <q-icon name="image" />
                </template>

                <template v-slot:append>
                  <q-icon name="search" @click.stop />
                </template>

                <template v-slot:hint> Tamaño máximo 2MB - 1600 x 1200 px. </template>
              </q-file>
              <q-btn class="q-my-xl" label="Cambiar Slider" @click="editProductSlider"></q-btn>
            </div>
    </div>
</div>  
</div>
</template>

<script>
import { api } from "../../boot/axios";

export default {
  name: "ProductsTable",
  created() {
    this.getProducts();
    this.getCategories();
    this.getSubcategories();
    this.getProductSlider();
  },
  data() {
    return {
      products: [],
      categories: [],
      subcategories: [],
      selectedCategories: [],
      selectedSubcategories: [],
      attachedProductImages: [],
      filter: '',
      activeCategoryFilter: false,
      subcategoryFilter: [],
      columns: [
        { name: "Id", label: "ID", field: "id", sortable: true, align: "left" },
        {
          name: "Image",
          label: "",
          field: "image_url",
          sortable: true,
          align: "left",
        },        
        {
          name: "Name",
          label: "Nombre",
          field: "name",
          sortable: true,
          align: "left",
        },
        {
          name: "Price",
          label: "Precio",
          field: "price",
          sortable: true,
          align: "left",
        },
        {
          name: "Description",
          label: "Descripción",
          field: "description",
          sortable: true,
          align: "left",
        },
        {
          name: "Active",
          label: "Activo",
          field: "active",
          sortable: true,
          align: "center",
        },
        {
          name: "Stock",
          label: "Stock",
          field: "stock",
          sortable: true,
          align: "center",
        },
        {
          name: "Action",
          label: "Acciones",
          field: "Action",
          sortable: true,
          align: "center",
        },
        {
          name: "Subcategories",
          label: "Subcategorias",
          field: "subcategories",
          align: "center",
        },
      ],
      visibleColumns: [ "Id", 'Name', "Price", "Description", "Active", "Stock", "Image", "Action"],
      pagination: {
        rowsPerPage: 5,
      },
      addProductDialog: false,
      relateProductDialog: false,
      editedIndex: -1,
      editedItem: {
        name: "",
        price: "",
        description: "",
        file: null,
        image_url: "",
        active: true,
        stock: true,
        product_images: [],
        additional_images: null,
      },
      defaultItem: {
        name: "",
        price: "",
        description: "",
        file: null,
        image_url: "",
        active: true,
        stock: true,
        product_images: [],
        additional_images: null,
      },
      productSection: [],
      productSlider: undefined,
    };
  },
  computed: {
    formTitle() {
      return this.editedIndex === -1 ? "Crear Producto" : "Editar Producto";
    },
  },
  methods: {
    getProducts() {
      api
        .get("products")
        .then((response) => {
          this.products = response.data;
        })
        .catch((e) => {
          console.log("error" + e);
        });
    },
    getCategories() {
      api
        .get("categories")
        .then((response) => {
          this.categories = response.data
        })
        .catch((e) => {
          console.log("error" + e);
        });
    },
    getSubcategories() {
      api
        .get("subcategories")
        .then((response) => {
            this.subcategories = response.data;
            this.subcategories = this.subcategories.map(subcategory => {
              subcategory['label'] = subcategory['name']
              subcategory['value'] = subcategory['id']
              delete subcategory['name']
              delete subcategory['id']
              return subcategory
            })
        })
        .catch((e) => {
          console.log("error" + e);
        });
    },
    getProductSlider() {
      api
        .get('internalSections')
        .then((response) => {
          this.productSection = response.data.slice(1,2)
        })
        .catch((e)=>{
          console.log('error' + e);
        })
    },
    editProduct(item) {
      this.editedIndex = this.products.indexOf(item);
      this.editedItem = Object.assign({}, item);

      if(this.products[this.editedIndex].product_images) {
        this.products[this.editedIndex].product_images.forEach(image => {
          this.attachedProductImages.push(image)
        })
      }
      
      this.addProductDialog = true;
    },
    deleteProduct(item) {
      const index = this.products.indexOf(item);
      const id = this.products[index].id;
      confirm("Estás seguro que querés eliminar este producto?") &&
        this.products.splice(index, 1) &&
        api.delete(`products/deleteProduct/${id}`).catch((e) => {
          console.log(e.response);
        });
    },
    relateProduct(item) {
      this.relateProductDialog = true;

      this.editedIndex = this.products.indexOf(item);
      this.products[this.editedIndex].categories.forEach(category => {
        this.selectedCategories.push(category)
      })
      this.products[this.editedIndex].subcategories.forEach(subcategory => {
        this.selectedSubcategories.push(subcategory)
      });
    },
    closeNewProductDialog() {
      this.addProductDialog = false;
      this.$nextTick(() => {
        this.editedItem = Object.assign({}, this.defaultItem);
        this.editedIndex = -1;
        this.attachedProductImages = []
      });
    },
    closeRelationDialog() {
      this.relateProductDialog = false;
      this.$nextTick(() => {
        this.selectedCategories = [];
        this.selectedSubcategories = [];
      });
    },
    saveNewProduct() {
      let self = this;

      let formData = new FormData();
      formData.set("name", this.editedItem.name);
      formData.set("price", this.editedItem.price);
      formData.set("description", this.editedItem.description);
      formData.set("active", this.editedItem.active)
      formData.set("stock", this.editedItem.stock)
      formData.append("file", this.editedItem.file);

      if (this.editedIndex > -1) {
        this.handleAttachedImagesForExistingProduct()
        let localIndex = this.editedIndex;
        api
          .put(`products/editProduct/${this.products[localIndex].id}`, formData)
          .then((data) => {
            Object.assign(this.products[localIndex], {
              id: data.data.id,
              name: data.data.name,
              price: data.data.price,
              description: data.data.description,
              file: null,
              image_url: data.data.image_url,
              active: data.data.active,
              stock: data.data.stock,
              additional_images: data.data.additional_images,
            });
            self.closeNewProductDialog()
          })
          .catch((e) => {
            console.log(e);
          });
      } else {
        
        api
          .post("products/addProduct", formData)
          .then((data) => {
            this.products.push({
              id: data.data.id,
              name: data.data.name,
              price: data.data.price,
              description: data.data.description,
              file: null,
              image_url: data.data.image_url,
              active: data.data.active,
              stock: data.data.stock,
              additional_images: data.data.additional_images,
            });
            self.handleAttachedImagesForNewProduct(data.data.id)
            self.closeNewProductDialog()
          })
          .catch((e) => {
            console.log(e.response.data.errors.message);
          });
      }
    },  
    handleAttachedImagesForExistingProduct() {
      let localIndex = this.editedIndex
      let self = this

      this.products[localIndex].product_images.forEach(image => {
        let found = this.attachedProductImages.some(el => {return el.id == image.id})
        if(!found) {
          api.post('products/separateImageFromProduct', {productId: this.products[localIndex].id, imageId: image.id })
          .then(() => {
            self.getProducts()
          })
        }
      })

      if(this.editedItem.additional_images != undefined) {
        this.editedItem.additional_images.forEach(image => {
        let imageFormData = new FormData();
        console.log(typeof this.products[localIndex].id)
        imageFormData.set("productId", this.products[localIndex].id)
        imageFormData.append("additional_images", image)

        

        api.post('products/relateImageToProduct', imageFormData)
        .then(() => {
          self.getProducts()
        })
      })
      }
      
    },
    handleAttachedImagesForNewProduct(newProductId) {
      let self = this

      if(this.editedItem.additional_images != undefined) {
        this.editedItem.additional_images.forEach(image => {
        let imageFormData = new FormData();
        imageFormData.set("productId", newProductId)
        imageFormData.append("additional_images", image)        

        api.post('products/relateImageToProduct', imageFormData)
        .then(() => {
          self.getProducts()
        })
      })
      } else {
        self.getProducts()
      }
      
    },
    setSelectedCategories(category) {
      if(this.selectedCategories.find(element => element.id == category.id) === undefined) {
        this.selectedCategories.push(category)
      }
    },
    setSelectedSubcategories(category, subcategory) {
      if(this.selectedCategories.find(element => element.id == category.id) === undefined) {
        this.selectedCategories.push(category)
      }
      if(this.selectedSubcategories.find(element => element.id == subcategory.id) === undefined) {
      this.selectedSubcategories.push(subcategory)
      }
    },
    unsetSelectedCategories(selectedCategory, selectedCategoryIndex) {
      this.selectedCategories.splice(selectedCategoryIndex,1)
      //Conocer las subcategorias de la categoria
      let categoryChilds = this.categories.find(category => category.id == selectedCategory.id).subcategories
      //Por cada una de las subcategorias, encontrarlas en las selectedSubcategories y eliminarlas
      categoryChilds.forEach(categoryChild => {
        let index = this.selectedSubcategories.indexOf(this.selectedSubcategories.find(selectedSubcategory => selectedSubcategory.id == categoryChild.id))
        if (index >= 0){
          this.selectedSubcategories.splice(index, 1)
          }
      })
    },
    saveNewSubcategoryRelation() {
      let localIndex = this.editedIndex
      // Primero, elimino todas las que estaban relacionadas, pero ahora no lo estan. Primero categorias, despues subcategorias
      this.products[localIndex].categories.forEach(category => {
        let deleted = this.selectedCategories.some(el => {return el.id == category.id})
        if(!deleted) {
          api.post('products/separateProductFromCategory', {productId: this.products[localIndex].id, categoryId: category.id })
        }
      })
      this.products[localIndex].subcategories.forEach(subcategory => {
        let deleted = this.selectedSubcategories.some(el => {return el.id == subcategory.id})
        if(!deleted) {
          api.post('products/separateProductFromSubcategory', {productId: this.products[localIndex].id, subcategoryId: subcategory.id })
        }
      })
      //Despues, asocio 
      this.selectedSubcategories.forEach(subcategory => {
        let found = this.products[localIndex].subcategories.some(el => {return el.id == subcategory.id})
        if(!found) {
          api.post('products/relateProductToSubcategory', {productId: this.products[localIndex].id, subcategoryId: subcategory.id })
        }
      })
      this.selectedCategories.forEach(category => {
        let found = this.products[localIndex].categories.some(el => {return el.id == category.id})
        if(!found) {
          api.post('products/relateProductToCategory', {productId: this.products[localIndex].id, categoryId: category.id })
        }
      })
      //Finalmente, vuelvo a pedir los productos (Verificar que todo funcione en orden y traiga los productos actualizadisimos)
      api
      .get("products")
      .then((response) => {
        this.products = response.data;
        this.closeRelationDialog()
      })
      .catch((e) => {
        console.log("error" + e);
      });
    },
    updateProductListWithCategory(category) {
      this.categories.forEach(category => {
        category.selected = false
        category.subcategories.forEach(subcategory => {
          subcategory.selected = false
        })
      })
      this.activeCategoryFilter = true
      api
        .get(`products/findForCategory/${category.id}`)
        .then((response) => {
          this.products = response.data;
        })
        .catch((e) => {
          console.log("error" + e);
        });
    },
    updateProductListWithSubcategory(subcategory) {
      this.categories.forEach(category => {
        category.selected = false
        category.subcategories.forEach(subcategory => {
          subcategory.selected = false
        })
      })
      this.activeCategoryFilter = true
      api
        .get(`products/findForSubcategory/${subcategory.id}`)
        .then((response) => {
          this.products = response.data;
        })
        .catch((e) => {
          console.log("error" + e);
        });
    },
    resetCategoryFilter() {
      api
      .get("products")
      .then((response) => {
        this.products = response.data
        this.activeCategoryFilter = false
        this.categories.forEach(category => {
        category.selected = false
        category.subcategories.forEach(subcategory => {
          subcategory.selected = false
        })
      })
      })
      .catch((e) => {
        console.log("error" + e);
      });
    },
    editProductSlider() {
      let self = this

      if(this.productSlider != undefined) {
        let imageFormData = new FormData();
        imageFormData.append("product_slider_image", this.productSlider)        

        api.put('internalSections/editProductSlider', imageFormData)
        .then(() => {
          self.getProductSlider()
        })
      } else {
        self.onNofileRejection()
      }
    },
    onRejected() {
      this.$q.notify({
        type: "negative",
        message: `Archivo no permitido`,
      });
    },
    onNofileRejection() {
      this.$q.notify({
        type: "negative",
        message: `Se debe seleccionar una imagen`,
      });
    },
  },
};
</script>

<style>
.pointer {
  cursor: pointer;
}

.selected {
  font-weight: 700;
  text-decoration-line: underline;
  text-decoration-color: cadetblue;
}
</style>
