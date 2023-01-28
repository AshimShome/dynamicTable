# dynamicTable

    Route::post('/product-service-return-store', 'ProductServiceReturnController@store')->name('product-service-return-store');

################################################################################################
  <div class="row" id="product_list">
        <form method="post" action="{{ route('product-service-return-store') }}">
            @csrf
            <input type="hidden" name="alias" value="purchase">
            <input type="hidden" name="alias_id" value="{{ $bill_details->bill_id }}">
            <div class="col-6">
                <table class="table">
                    <thead>
                    <tr>
                        <th scope="col">Product</th>
                        <th scope="col">Return Quentity</th>
                        <th scope="col">Price</th>
                        <th scope="col">Amount</th>
                    </tr>
                    </thead>
                    <tbody id="product_list_body">
                    </tbody>
                </table>
            </div>
            <div class="col-6 text-right">
                <button type="submit" class="btn btn-success">Submit</button>
            </div>
        </form>
    </div>

##########################################################################################################
public function store(Request $request)
    {
        try {
            $returnProducts = $request->all();
//            dd($returnProducts['product_id']);
            if (count($returnProducts['bill_product_id']) > 0) {
                for ($indx = 0; $indx < count($returnProducts['bill_product_id']); $indx++) {
                    $purchaseReturn = new ProductServiceReturn();
                    $purchaseReturn->alias = $returnProducts['alias'];
                    $purchaseReturn->alias_id = $returnProducts['alias_id'];
                    $purchaseReturn->product_id = $returnProducts['product_id'][$indx];
                    $purchaseReturn->return_quentity = $returnProducts['return_quantity'][$indx];
                    $purchaseReturn->save();
                }
            }

            return redirect()->route('product-service-return-index');
        } catch (Exception $exception) {
            Log::debug("Failed to store purchase return. " . $exception->getMessage());
            return redirect()->back();
        }
    }
    
    ##########################################################################################################

@push('script-page')
    <script src="https://code.jquery.com/jquery-3.6.3.min.js"
            integrity="sha256-pvPw+upLPUjgMXY0G+8O0xUf+/Im1MZjXxxgOcBQBXU=" crossorigin="anonymous">
    </script>
    <script>
        $(function () {
            var addedItems = [];
            $(document).on('click', '.product-items', function () {
                let rowDt = [];
                rowDt['itemId'] = $(this).val();
                rowDt['prodId'] = $(this).closest('tr').find('.prod-id').text();
                rowDt['prodName'] = $(this).closest('tr').find('.prod-name').text();
                rowDt['prodQty'] = $(this).closest('tr').find('.prod-quantity').text();
                rowDt['prodPrice'] = $(this).closest('tr').find('.prod-price').text();
                rowDt['totalAmount'] = parseInt(rowDt['prodQty']) * parseInt(rowDt['prodPrice']);

                // find current row id
                if (addedItems.length > 0 && addedItems.includes(rowDt['itemId'])) {
                    $(`.item-row-id-${rowDt['itemId']}`).closest('tr').remove();
                    const index = addedItems.indexOf(rowDt['itemId']);
                    addedItems.splice(index, 1);
                } else {
                    addItems(rowDt);
                    addedItems.push(rowDt['itemId']);
                }
            });

            function addItems(rowDt) {
                $('#product_list_body').append(
                    `<tr>
                    <td class="item-row-id-${rowDt['itemId']}" style="display: none;"><input type="number" name="bill_product_id[]" class="form-control" value="${rowDt['itemId']}"></td>
                    <td style="display: none;"><input type="hidden" name="product_id[]" class="form-control" value="${rowDt['prodId']}"></td>
                    <td>${rowDt['prodName']}</td>
                    <td><input type="number" name="return_quantity[]" class="form-control" value="${rowDt['prodQty']}"></td>
                    <td>${rowDt['prodPrice']}</td>
                    <td>${rowDt['totalAmount']}</td>
                    </tr>`
                );
            }
        });

    </script>
@endpush

    ##########################################################################################################
