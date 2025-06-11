<template>
  <div id="main-container">
    <div class="wrapper">
      <div class="top-bar">
        <div class="logo-area">
          <div class="logo-text">ショッピングモール</div>
        </div>
        <div class="search-area">
          <div class="search-box">
            <div class="search-input" contenteditable="true" @input="handleSearch">商品を検索...</div>
            <div class="search-button">検索</div>
          </div>
        </div>
        <div class="user-area">
          <div class="cart-icon">
            <div class="cart-count">{{ cartItems }}</div>
            <div class="cart-text">カート</div>
          </div>
          <div class="user-menu">
            <div class="user-name">ゲスト様</div>
          </div>
        </div>
      </div>

      <div class="category-bar">
        <div v-for="category in categories" :key="category" class="category-item">
          <div class="category-link" 
               :class="{ active: selectedCategory === category }"
               @click="selectCategory(category)">{{ category }}</div>
        </div>
      </div>

      <div class="main-content">
        <div class="sidebar">
          <div class="filter-section">
            <div class="filter-title">絞り込み</div>
            <div class="filter-group">
              <div class="filter-label">価格帯</div>
              <div v-for="range in priceRanges" :key="range" class="filter-option" @click="togglePriceRange(range)">
                <div class="checkbox" :class="{ checked: selectedPriceRanges.includes(range) }">
                  <div v-if="selectedPriceRanges.includes(range)" class="checkmark">✓</div>
                </div>
                <div class="filter-text">{{ range }}</div>
              </div>
            </div>
            <div class="filter-group">
              <div class="filter-label">ブランド</div>
              <div v-for="brand in brands" :key="brand" class="filter-option" @click="toggleBrand(brand)">
                <div class="checkbox" :class="{ checked: selectedBrands.includes(brand) }">
                  <div v-if="selectedBrands.includes(brand)" class="checkmark">✓</div>
                </div>
                <div class="filter-text">{{ brand }}</div>
              </div>
            </div>
            <div class="filter-actions">
              <div class="clear-filters" @click="clearFilters">
                <div class="clear-text">フィルターをクリア</div>
              </div>
            </div>
          </div>
        </div>

        <div class="product-area">
          <div class="sort-bar">
            <div class="result-count">{{ products.length }}件の商品</div>
            <div class="sort-options">
              <div class="sort-label">並び替え:</div>
              <div class="sort-select" @click="toggleSort">
                <div class="sort-value">{{ currentSort }}</div>
                <div class="sort-arrow">▼</div>
              </div>
            </div>
          </div>

          <div class="products-grid">
            <div v-for="product in displayProducts" :key="`product-${product.id}-${product.sortKey}`" 
                 class="product-card">
              <div class="product-image-wrapper">
                <div class="product-image" :style="{ backgroundColor: product.color }">
                  <div class="image-placeholder">{{ product.name.substring(0, 2) }}</div>
                </div>
                <div v-if="product.stock < 5" class="stock-badge">
                  <div class="badge-text">残り{{ product.stock }}個</div>
                </div>
              </div>
              <div class="product-info">
                <div class="product-name">{{ product.name }}</div>
                <div class="product-description">{{ product.description }}</div>
                <div class="product-meta">
                  <div class="price-section">
                    <div class="price-current">¥{{ product.price.toLocaleString() }}</div>
                    <div v-if="product.originalPrice" class="price-original">
                      ¥{{ product.originalPrice.toLocaleString() }}
                    </div>
                  </div>
                  <div class="rating-section">
                    <div class="stars">
                      <div v-for="n in 5" :key="n" class="star" 
                           :class="{ filled: n <= product.rating }">★</div>
                    </div>
                    <div class="review-count">({{ product.reviews }})</div>
                  </div>
                </div>
                <div class="product-actions">
                  <div class="add-to-cart" @click="addToCart(product)">
                    <div class="cart-icon-small">🛒</div>
                    <div class="add-text">カートに追加</div>
                  </div>
                  <div class="favorite-button" @click="toggleFavorite(product)">
                    <div class="heart-icon">{{ product.favorited ? '❤️' : '🤍' }}</div>
                  </div>
                </div>
              </div>
            </div>
          </div>

          <div class="pagination">
            <div class="page-button">前へ</div>
            <div v-for="page in 5" :key="page" class="page-number" 
                 :class="{ active: page === currentPage }">{{ page }}</div>
            <div class="page-button">次へ</div>
          </div>
        </div>
      </div>

      <div class="promotion-banner">
        <div class="banner-content">
          <div class="banner-title">期間限定セール実施中！</div>
          <div class="banner-subtitle">全品最大50%OFF</div>
          <div class="banner-timer">残り時間: {{ saleTimer }}</div>
        </div>
      </div>

      <div class="footer">
        <div class="footer-content">
          <div class="footer-column">
            <div class="footer-title">ショッピングガイド</div>
            <div class="footer-link">ご利用方法</div>
            <div class="footer-link">送料について</div>
            <div class="footer-link">返品・交換</div>
          </div>
          <div class="footer-column">
            <div class="footer-title">カスタマーサポート</div>
            <div class="footer-link">お問い合わせ</div>
            <div class="footer-link">よくある質問</div>
            <div class="footer-link">配送状況確認</div>
          </div>
          <div class="footer-column">
            <div class="footer-title">会社情報</div>
            <div class="footer-link">会社概要</div>
            <div class="footer-link">採用情報</div>
            <div class="footer-link">プレスリリース</div>
          </div>
        </div>
        <div class="footer-bottom">
          <div class="copyright">© 2024 ショッピングモール All Rights Reserved.</div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'App',
  data() {
    return {
      cartItems: 0,
      currentPage: 1,
      currentSort: 'おすすめ順',
      saleTimer: '',
      categories: ['すべて', 'ファッション', '家電', '食品', 'インテリア', '本・雑誌', 'スポーツ', 'おもちゃ'],
      selectedCategory: 'すべて',
      priceRanges: ['〜¥1,000', '¥1,000〜¥5,000', '¥5,000〜¥10,000', '¥10,000〜'],
      selectedPriceRanges: [],
      brands: ['ブランドA', 'ブランドB', 'ブランドC', 'ブランドD', 'ブランドE'],
      selectedBrands: [],
      products: [],
      displayProducts: [],
      productColors: ['#E8E8E8', '#F0E6E6', '#E6F0F0', '#F0F0E6', '#E6E6F0', '#F0E6F0', '#E6F0E6']
    }
  },
  mounted() {
    this.initializeProducts()
    this.randomizeDisplay()
    this.startSaleTimer()
    this.startStockUpdates()
  },
  methods: {
    initializeProducts() {
      const productData = [
        { name: 'ワイヤレスイヤホン', category: '家電', brand: 'ブランドA' },
        { name: 'スマートウォッチ', category: '家電', brand: 'ブランドB' },
        { name: 'ノートパソコン', category: '家電', brand: 'ブランドC' },
        { name: 'コーヒーメーカー', category: '家電', brand: 'ブランドD' },
        { name: 'ヨガマット', category: 'スポーツ', brand: 'ブランドE' },
        { name: 'ランニングシューズ', category: 'スポーツ', brand: 'ブランドA' },
        { name: 'バックパック', category: 'ファッション', brand: 'ブランドB' },
        { name: 'デスクライト', category: 'インテリア', brand: 'ブランドC' },
        { name: 'ワイヤレスマウス', category: '家電', brand: 'ブランドD' },
        { name: 'USBハブ', category: '家電', brand: 'ブランドE' },
        { name: 'モバイルバッテリー', category: '家電', brand: 'ブランドA' },
        { name: 'Bluetoothスピーカー', category: '家電', brand: 'ブランドB' },
        { name: 'タブレットスタンド', category: '家電', brand: 'ブランドC' },
        { name: 'キーボード', category: '家電', brand: 'ブランドD' },
        { name: '料理本', category: '本・雑誌', brand: 'ブランドE' },
        { name: 'インテリア雑誌', category: '本・雑誌', brand: 'ブランドA' },
        { name: 'おもちゃブロック', category: 'おもちゃ', brand: 'ブランドB' },
        { name: 'ぬいぐるみ', category: 'おもちゃ', brand: 'ブランドC' },
        { name: 'オーガニック野菜セット', category: '食品', brand: 'ブランドD' },
        { name: 'コーヒー豆', category: '食品', brand: 'ブランドE' }
      ]
      
      const descriptions = [
        '高品質な商品です', '人気の定番アイテム', '今季のトレンド商品', '限定セール中',
        'レビュー高評価', 'ベストセラー商品', '新商品', 'お買い得品'
      ]
      
      this.products = productData.map((data, index) => ({
        id: index,
        name: data.name,
        category: data.category,
        brand: data.brand,
        description: descriptions[Math.floor(Math.random() * descriptions.length)],
        price: Math.floor(Math.random() * 50000) + 500,
        originalPrice: Math.random() > 0.5 ? Math.floor(Math.random() * 70000) + 5000 : null,
        stock: Math.floor(Math.random() * 20) + 1,
        rating: Math.floor(Math.random() * 3) + 3,
        reviews: Math.floor(Math.random() * 500) + 10,
        color: this.productColors[Math.floor(Math.random() * this.productColors.length)],
        favorited: false,
        sortKey: Math.random()
      }))
    },
    randomizeDisplay() {
      this.applyFilters()
    },
    applyFilters() {
      let filtered = [...this.products]
      
      // カテゴリフィルター
      if (this.selectedCategory !== 'すべて') {
        filtered = filtered.filter(product => product.category === this.selectedCategory)
      }
      
      // 価格帯フィルター
      if (this.selectedPriceRanges.length > 0) {
        filtered = filtered.filter(product => {
          return this.selectedPriceRanges.some(range => {
            switch(range) {
              case '〜¥1,000':
                return product.price <= 1000
              case '¥1,000〜¥5,000':
                return product.price > 1000 && product.price <= 5000
              case '¥5,000〜¥10,000':
                return product.price > 5000 && product.price <= 10000
              case '¥10,000〜':
                return product.price > 10000
              default:
                return true
            }
          })
        })
      }
      
      // ブランドフィルター
      if (this.selectedBrands.length > 0) {
        filtered = filtered.filter(product => this.selectedBrands.includes(product.brand))
      }
      
      // ソート適用
      switch(this.currentSort) {
        case '価格が安い順':
          filtered.sort((a, b) => a.price - b.price)
          break
        case '価格が高い順':
          filtered.sort((a, b) => b.price - a.price)
          break
        case '評価順':
          filtered.sort((a, b) => b.rating - a.rating)
          break
        case '新着順':
          filtered.sort(() => Math.random() - 0.5)
          break
        default:
          filtered.sort(() => Math.random() - 0.5)
      }
      
      // 商品を更新
      this.displayProducts = filtered.map(product => ({
        ...product,
        stock: Math.floor(Math.random() * 20) + 1,
        sortKey: Math.random()
      }))
    },
    selectCategory(category) {
      this.selectedCategory = category
      this.applyFilters()
    },
    togglePriceRange(range) {
      const index = this.selectedPriceRanges.indexOf(range)
      if (index > -1) {
        this.selectedPriceRanges.splice(index, 1)
      } else {
        this.selectedPriceRanges.push(range)
      }
      this.applyFilters()
    },
    toggleBrand(brand) {
      const index = this.selectedBrands.indexOf(brand)
      if (index > -1) {
        this.selectedBrands.splice(index, 1)
      } else {
        this.selectedBrands.push(brand)
      }
      this.applyFilters()
    },
    clearFilters() {
      this.selectedCategory = 'すべて'
      this.selectedPriceRanges = []
      this.selectedBrands = []
      this.applyFilters()
    },
    toggleSort() {
      const sorts = ['おすすめ順', '価格が安い順', '価格が高い順', '新着順', '評価順']
      const currentIndex = sorts.indexOf(this.currentSort)
      this.currentSort = sorts[(currentIndex + 1) % sorts.length]
      this.applyFilters()
    },
    addToCart(product) {
      this.cartItems++
      product.stock = Math.max(0, product.stock - 1)
    },
    toggleFavorite(product) {
      product.favorited = !product.favorited
    },
    handleSearch(event) {
      // 検索時に商品を再シャッフル
      if (event.target.textContent.length > 2) {
        this.applyFilters()
      }
    },
    startSaleTimer() {
      const updateTimer = () => {
        const now = new Date()
        const hours = 23 - now.getHours()
        const minutes = 59 - now.getMinutes()
        const seconds = 59 - now.getSeconds()
        this.saleTimer = `${hours}時間${minutes}分${seconds}秒`
      }
      
      updateTimer()
      setInterval(updateTimer, 1000)
    },
    startStockUpdates() {
      // 定期的に在庫数を変更
      setInterval(() => {
        const randomProduct = this.displayProducts[Math.floor(Math.random() * this.displayProducts.length)]
        if (randomProduct) {
          randomProduct.stock = Math.floor(Math.random() * 20) + 1
        }
      }, 5000)
    }
  }
}
</script>

<style scoped>
#main-container {
  min-height: 100vh;
  background-color: #f5f5f5;
}

.wrapper {
  max-width: 1400px;
  margin: 0 auto;
}

.top-bar {
  background: white;
  padding: 15px 20px;
  display: flex;
  align-items: center;
  box-shadow: 0 2px 4px rgba(0,0,0,0.1);
  position: sticky;
  top: 0;
  z-index: 100;
}

.logo-area {
  flex: 0 0 200px;
}

.logo-text {
  font-size: 24px;
  font-weight: bold;
  color: #333;
}

.search-area {
  flex: 1;
  padding: 0 30px;
}

.search-box {
  display: flex;
  background: #f0f0f0;
  border-radius: 25px;
  overflow: hidden;
}

.search-input {
  flex: 1;
  padding: 10px 20px;
  background: transparent;
  border: none;
  outline: none;
  color: #666;
}

.search-button {
  padding: 10px 25px;
  background: #ff6b6b;
  color: white;
  cursor: pointer;
  transition: background 0.3s;
}

.search-button:hover {
  background: #ff5252;
}

.user-area {
  display: flex;
  align-items: center;
  gap: 25px;
}

.cart-icon {
  position: relative;
  cursor: pointer;
}

.cart-count {
  position: absolute;
  top: -8px;
  right: -8px;
  background: #ff6b6b;
  color: white;
  width: 20px;
  height: 20px;
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 12px;
  font-weight: bold;
}

.cart-text {
  margin-top: 5px;
  font-size: 14px;
}

.user-name {
  font-size: 14px;
  color: #666;
}

.category-bar {
  background: white;
  padding: 10px 20px;
  display: flex;
  gap: 30px;
  border-bottom: 1px solid #eee;
  overflow-x: auto;
}

.category-item {
  flex-shrink: 0;
}

.category-link {
  padding: 8px 16px;
  cursor: pointer;
  transition: color 0.3s;
  font-size: 14px;
}

.category-link:hover {
  color: #ff6b6b;
}

.category-link.active {
  color: #ff6b6b;
  font-weight: bold;
  border-bottom: 2px solid #ff6b6b;
}

.main-content {
  display: flex;
  gap: 20px;
  padding: 20px;
}

.sidebar {
  flex: 0 0 250px;
  background: white;
  padding: 20px;
  border-radius: 8px;
  height: fit-content;
}

.filter-section {
  
}

.filter-title {
  font-size: 18px;
  font-weight: bold;
  margin-bottom: 20px;
}

.filter-group {
  margin-bottom: 25px;
}

.filter-label {
  font-weight: bold;
  margin-bottom: 10px;
  font-size: 14px;
}

.filter-option {
  display: flex;
  align-items: center;
  gap: 10px;
  padding: 8px 0;
  cursor: pointer;
}

.checkbox {
  width: 18px;
  height: 18px;
  border: 2px solid #ddd;
  border-radius: 3px;
  display: flex;
  align-items: center;
  justify-content: center;
  transition: all 0.3s;
}

.checkbox.checked {
  background: #ff6b6b;
  border-color: #ff6b6b;
}

.checkmark {
  color: white;
  font-size: 12px;
  font-weight: bold;
}

.filter-text {
  font-size: 14px;
  color: #666;
}

.filter-actions {
  margin-top: 30px;
  padding-top: 20px;
  border-top: 1px solid #eee;
}

.clear-filters {
  padding: 8px 15px;
  background: #f0f0f0;
  border-radius: 5px;
  cursor: pointer;
  transition: background 0.3s;
  text-align: center;
}

.clear-filters:hover {
  background: #e0e0e0;
}

.clear-text {
  font-size: 14px;
  color: #666;
}

.product-area {
  flex: 1;
}

.sort-bar {
  background: white;
  padding: 15px 20px;
  display: flex;
  justify-content: space-between;
  align-items: center;
  border-radius: 8px;
  margin-bottom: 20px;
}

.result-count {
  font-size: 16px;
  color: #666;
}

.sort-options {
  display: flex;
  align-items: center;
  gap: 10px;
}

.sort-label {
  font-size: 14px;
  color: #666;
}

.sort-select {
  display: flex;
  align-items: center;
  gap: 5px;
  padding: 8px 15px;
  background: #f0f0f0;
  border-radius: 5px;
  cursor: pointer;
}

.sort-value {
  font-size: 14px;
}

.sort-arrow {
  font-size: 12px;
}

.products-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(280px, 1fr));
  gap: 20px;
  margin-bottom: 40px;
}

.product-card {
  background: white;
  border-radius: 8px;
  overflow: hidden;
  transition: transform 0.3s, box-shadow 0.3s;
  cursor: pointer;
}

.product-card:hover {
  transform: translateY(-5px);
  box-shadow: 0 5px 20px rgba(0,0,0,0.15);
}

.product-image-wrapper {
  position: relative;
  height: 200px;
  overflow: hidden;
}

.product-image {
  width: 100%;
  height: 100%;
  display: flex;
  align-items: center;
  justify-content: center;
}

.image-placeholder {
  font-size: 48px;
  color: #999;
  font-weight: bold;
}

.stock-badge {
  position: absolute;
  top: 10px;
  right: 10px;
  background: #ff6b6b;
  color: white;
  padding: 5px 10px;
  border-radius: 15px;
}

.badge-text {
  font-size: 12px;
  font-weight: bold;
}

.product-info {
  padding: 15px;
}

.product-name {
  font-size: 16px;
  font-weight: bold;
  margin-bottom: 5px;
}

.product-description {
  font-size: 14px;
  color: #666;
  margin-bottom: 10px;
}

.product-meta {
  display: flex;
  justify-content: space-between;
  align-items: flex-start;
  margin-bottom: 15px;
}

.price-section {
  
}

.price-current {
  font-size: 20px;
  font-weight: bold;
  color: #ff6b6b;
}

.price-original {
  font-size: 14px;
  color: #999;
  text-decoration: line-through;
}

.rating-section {
  text-align: right;
}

.stars {
  display: flex;
  gap: 2px;
}

.star {
  color: #ddd;
  font-size: 14px;
}

.star.filled {
  color: #ffd700;
}

.review-count {
  font-size: 12px;
  color: #666;
}

.product-actions {
  display: flex;
  gap: 10px;
}

.add-to-cart {
  flex: 1;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 8px;
  padding: 10px;
  background: #ff6b6b;
  color: white;
  border-radius: 5px;
  cursor: pointer;
  transition: background 0.3s;
}

.add-to-cart:hover {
  background: #ff5252;
}

.cart-icon-small {
  font-size: 16px;
}

.add-text {
  font-size: 14px;
  font-weight: bold;
}

.favorite-button {
  display: flex;
  align-items: center;
  justify-content: center;
  width: 40px;
  height: 40px;
  border: 1px solid #ddd;
  border-radius: 5px;
  cursor: pointer;
  transition: border-color 0.3s;
}

.favorite-button:hover {
  border-color: #ff6b6b;
}

.heart-icon {
  font-size: 20px;
}

.pagination {
  display: flex;
  justify-content: center;
  align-items: center;
  gap: 10px;
  margin: 40px 0;
}

.page-button, .page-number {
  padding: 8px 15px;
  background: white;
  border: 1px solid #ddd;
  border-radius: 5px;
  cursor: pointer;
  font-size: 14px;
  transition: all 0.3s;
}

.page-button:hover, .page-number:hover {
  border-color: #ff6b6b;
  color: #ff6b6b;
}

.page-number.active {
  background: #ff6b6b;
  color: white;
  border-color: #ff6b6b;
}

.promotion-banner {
  background: linear-gradient(135deg, #ff6b6b 0%, #ff5252 100%);
  color: white;
  padding: 40px;
  text-align: center;
  margin: 40px 20px;
  border-radius: 8px;
}

.banner-title {
  font-size: 28px;
  font-weight: bold;
  margin-bottom: 10px;
}

.banner-subtitle {
  font-size: 18px;
  margin-bottom: 15px;
}

.banner-timer {
  font-size: 24px;
  font-weight: bold;
}

.footer {
  background: #333;
  color: white;
  padding: 40px 20px 20px;
  margin-top: 60px;
}

.footer-content {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 40px;
  max-width: 1000px;
  margin: 0 auto;
  margin-bottom: 30px;
}

.footer-column {
  
}

.footer-title {
  font-size: 16px;
  font-weight: bold;
  margin-bottom: 15px;
}

.footer-link {
  font-size: 14px;
  color: #ccc;
  padding: 5px 0;
  cursor: pointer;
  transition: color 0.3s;
}

.footer-link:hover {
  color: white;
}

.footer-bottom {
  text-align: center;
  padding-top: 20px;
  border-top: 1px solid #555;
}

.copyright {
  font-size: 14px;
  color: #999;
}
</style>