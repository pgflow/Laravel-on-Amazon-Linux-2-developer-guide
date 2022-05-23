
    @php
    $category_id = array();
    $category_name = array();
    @endphp

    @foreach($view_category as $category)
        @php
        $category_remaining = $loop->remaining;
        $category_id += array($loop->remaining => $category->category);
        $category_name += array($category->category => array());
        @endphp
            @foreach($Me_stock_history as $stockinfo)
                @if($category_id[$category_remaining] === $stockinfo->category)
                @php
                $category = $stockinfo['category'];
                $category_name[$category] += array($loop->remaining => $stockinfo);
                @endphp
                @endif
            @endforeach
    @endforeach

    @foreach($category_id as $id)
        @foreach($category_name[$id] as $stockinfo)

        @if($loop->first)
        {{$id}}<br>
        @endif

        <strong>{{$loop->remaining}}配列の番号</strong>{{$stockinfo->jan}}<br>

        @if($loop->last and $loop->even)
        ループの最後で偶数<br>
        @endif

        @if($loop->last and $loop->odd)
        ループの最後で奇数<br>
        @endif

        @endforeach

    @endforeach