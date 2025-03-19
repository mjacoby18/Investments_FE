<template>
    <div class="container-fluid">
        <div class="row align-items-center justify-content-center">
            <div class="col col-12 align-items-center justify-content-center">
                <blockquote>
                    Welcome {{ validUserName }}!
                    <footer>
                        <small>
                            <em>&mdash;Financial Services, your Midwest Financial Services Partner.</em>
                        </small>
                    </footer>
                </blockquote>
            </div>
            <div class="col-12 col-md-10 col-lg-10 col-12 align-items-center justify-content-center">
                <div class="alert alert-success" v-if="showMsg === 'new'">New Stock has been added.</div>
                <div class="alert alert-success" v-if="showMsg === 'update'">Stock information has been updated.</div>
                <div class="alert alert-success" v-if="showMsg === 'deleted'">Selected stock has been deleted.</div>
            </div>
        </div>
        <div class="col col-12 col-md-10">
            <table class="table table-hover">
                <thead>
                    <tr>
                        <th scope="col">Customer</th>
                        <th scope="col">Stock Symbol</th>
                        <th scope="col">Company Name</th>
                        <th scope="col">Shares</th>
                        <th scope="col">Purchase Price</th>
                        <th scope="col">Purchase Date</th>
                        <th scope="col">Update</th>
                        <th scope="col">Delete</th>
                    </tr>
                </thead>
                <tbody>
                    <tr v-for="stock in stocks" :key="stock.pk">
                        <td>{{ stock.customer }}</td>
                        <td>{{ stock.symbol }}</td>
                        <td>{{ stock.name }}</td>
                        <td>{{ stock.shares }}</td>
                        <td>${{ stock.purchase_price }}</td>
                        <td>{{ stock.purchase_date }}</td>
                        <td><button @click="updateStock(stock)">Edit</button></td>
                        <td><button @click="deleteStock(stock)">Delete</button></td>
                    </tr>
                </tbody>
            </table>
            <button type="button" class="btn btn-primary" @click="addNewStock">Add New Stock</button>
        </div>
    </div>
  </template>
  
  <script>
  import router from '../router';
  import { APIService } from '../http/APIService';
  const apiService = new APIService();
  
  export default {
    name: "StockList",
    data() {
        return {
            stocks: [],
            validUserName: "Guest",
            showMsg: '',
            authenticated: false
        };
    },
    mounted() {
        this.authenticated = localStorage.getItem("isAuthenticated");
        this.getStocks();
        this.showMessages();
    },
    methods: {
        showMessages() {
            if (this.$route.params.msg) {
                this.showMsg = this.$route.params.msg;
            }
        },
        getStocks() {
            apiService.getStockList().then(response => {
                this.stocks = response.data.data;
                console.log("Stock data received:", this.stocks);
                if (localStorage.getItem("isAuthenticated") && JSON.parse(localStorage.getItem("isAuthenticated")) === true) {
                    this.validUserName = JSON.parse(localStorage.getItem("log_user"));
                }
            }).catch(error => {
                if (error.response.status === 401) {
                    localStorage.clear();
                    router.push("/auth");
                }
            });
        },
        addNewStock() {
            if (localStorage.getItem("isAuthenticated") && JSON.parse(localStorage.getItem("isAuthenticated")) === true) {
                router.push('/stock-create');
            } else {
                router.push("/auth");
            }
        },
        updateStock(stock) {
            router.push('/stock-create/' + stock.pk);
        },
        deleteStock(stock) {
            if (confirm("Do you really want to delete?")) {
                apiService.deleteStock(stock.pk).then(response => {
                    if (response.status === 204) {
                        router.push('/stock-list/deleted/');
                        this.getStocks();
                    }
                }).catch(error => {
                    if (error.response.status === 401) {
                        localStorage.clear();
                        router.push("/auth");
                    }
                });
            }
        }
    }
  };
  </script>
  
  <style>
    button {
        padding: 1rem;
        border: 0;
        cursor: pointer;
    }
  </style>
  