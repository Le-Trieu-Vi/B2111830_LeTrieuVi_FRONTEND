<template>
	<div class="page row">
		<div class="col-md-10">
			<InputSearch v-model="searchText" />
		</div>
		<div class="mt-3 col-md-6">
			<h4>
				Các liên hệ yêu thích
				<i class="fa-solid fa-heart-circle-check"></i>
			</h4>
			<ContactList v-if="filteredContactsCount > 0" :contacts="filteredContacts" v-model:activeIndex="activeIndex" />
			<p v-else>Không có liên hệ nào.</p>

			<div class="mt-3 row justify-content-around align-items-center">
				<button class="btn btn-sm btn-primary" @click="refreshList()">
                    <i class="fa-solid fa-rotate-right"></i> Làm mới
				</button>

				<button class="btn btn-sm btn-success" @click="goToAddContact">
					<i class="fas fa-plus"></i> Thêm mới
				</button>

                <button class="btn btn-sm btn-danger" @click="removeAllFavoriteContacts">
				    <i class="fas fa-trash"></i>&nbsp;Xóa tất cả
				</button>

			</div>
			<p class="message">{{ message }}</p>
		</div>
		<div class="mt-3 col-md-6">
			<div v-if="activeContact">
				<h4>
					Chi tiết Liên hệ
					<i class="fas fa-address-card"></i>
				</h4>
				<ContactCard :contact="activeContact"/>
				<router-link
					:to="{
						name: 'contact.edit',
						params: { id: activeContact._id },
					}"
				>
					<span class="mt-2 badge badge-warning">
						<i class="fas fa-edit"></i> Hiệu chỉnh
					</span>
				</router-link>
			</div>
		</div>
	</div>
</template>

<script>
import ContactCard from '@/components/ContactCard.vue';
import InputSearch from '@/components/InputSearch.vue';
import ContactList from '@/components/ContactList.vue';
import ContactService from '@/services/contact.service';

export default {
	components: {
		ContactCard,
		InputSearch,
		ContactList,
	},
	data() {
		return {
			contacts: [],
			activeIndex: -1,
			searchText: "",
			message: "",
		};
	},
	watch: {
		// Giám sát các thay đổi của biến searchText.
		// Bỏ chọn phần tử đang được chọn trong danh sách.
		searchText() {
			this.activeIndex = -1;
		},
	},
	computed: {
		// Chuyển các đối tượng contact thành chuỗi để tiện cho tìm kiếm.
		contactFavoriteStrings() {
			return this.contacts.map((contact) => {
				const { name, email, address, phone } = contact;
				return [name, email, address, phone].join("");
			});
		},
		// Trả về các contact có chứa thông tin cần tìm kiếm.
		filteredContacts() {
			if (!this.searchText) return this.contacts;
			return this.contacts.filter((_contact, index) =>
				this.contactFavoriteStrings[index].toLowerCase().includes(this.searchText)
			);
		},
		activeContact() {
			if (this.activeIndex < 0) return null;
			return this.filteredContacts[this.activeIndex];
		},
		filteredContactsCount() {
			return this.filteredContacts.length;
		},
	},
	methods: {
		async retrieveFavoriteContacts() {
			try {
				this.contacts = await ContactService.getAllFavorite();
			} catch (error) {
				console.log(error);
			}
		},

		refreshList() {
			this.retrieveFavoriteContacts();
			this.activeIndex = -1;
		},

		async removeAllFavoriteContacts() {
			if (confirm("Bạn muốn xóa tất cả liên hệ yêu thích?")) {
				try {
					// await ContactService.deleteAll();
					for (let i=0; i<this.filteredContactsCount; i++) {
						let id = this.filteredContacts[i]._id;
						await ContactService.delete(id);
						this.message = 'Xoá các liên hệ yêu thích thành công!';
					}
					this.refreshList();
				} catch (error) {
					console.log(error);
				}
			}
		},

		goToAddContact() {
			this.$router.push({ name: "contact.add" });
		},
	},
	mounted() {
		this.refreshList();
	},
}
</script>

<style scoped>
.page {
	text-align: left;
	max-width: 750px;
	margin: 0 auto;
}

.btn:hover {
    border: 1px solid;
    box-shadow: inset 0 0 20px rgba(255, 255, 255, .5), 0 0 20px rgba(255, 255, 255, .2);
    outline-color: rgba(255, 255, 255, 0);
    outline-offset: 15px;
    text-shadow: 1px 1px 2px #427388;
}
</style>