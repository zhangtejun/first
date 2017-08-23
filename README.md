/**
 * Created by zhangtj on 2017/8/23.
 */
define([ 'app', 'component', 'directive', 'service' ], function(app) {
    app.controller('QueryEntrustCtrl', function($scope, service, $state, service, $ionicScrollDelegate, $timeout) {
        service.setTitle("交易查询");
        $scope.tabIndex = 0;
        $scope.qryIndex = 0;

        $scope.BeginDate = "";
        $scope.EndDate = "";

        // 期限查询 0：当日，1：七日，2：30日，3：更多
        $scope.qryCheck = function(index) {
            $ionicScrollDelegate.resize();//重新计算它的容器大小
            $scope.currentPage = 0;// 清空
            $scope.LoopResult = null;//重置结果
            $scope.errorMsg = null;//重置无记录提示信息
            $scope.qryIndex = index;

            //默认日期为当日
            var sendData = {
                BeginDate: service.GetDate(0).BeginDate,
                EndDate: service.GetDate(0).EndDate
            }

            switch (index) {
                case "0":
                    $scope.QueryAll(sendData);
                    break;
                case "1":
                    sendData.BeginDate = service.GetDate(7).BeginDate;
                    sendData.EndDate = service.GetDate(7).EndDate;
                    $scope.QueryAll(sendData);
                    break;
                case "2":
                    sendData.BeginDate = service.GetDate(30).BeginDate;
                    sendData.EndDate = service.GetDate(30).EndDate;
                    $scope.QueryAll(sendData);
                    break;
                case "3":
                    WebViewJavascriptBridge.callHandler('callDatePicker', "", function (response) {
                        response = angular.fromJson(response);
                        //$scope.BeginDateR=response.startDate;
                        //$scope.EndDateR=response.endDate;
                        //$scope.BeginDate=response.startDate;
                        //$scope.EndDate=response.endDate;
                        sendData.BeginDate = response.startDate;
                        sendData.EndDate = response.endDate;
                        $scope.QueryAll(sendData);
                    });
                    break;
            }

        }//期限查询 END


        /** tab切换 **/
        $scope.tabCheck = function(index) {
            $scope.currentPage = 0;// 清空分页标识
            $scope.LoopResult = null;
            $scope.errorMsg = null;
            $scope.tabIndex = index;//重置tabIndex,如果需要。
            $scope.qryIndex = 0;//tab切换清空期限查询标识qryIndex，如果需要。
            //doSomething
        }

        /** 执行查询**/
        $scope.QueryAll = function(sendData, param) {
            //doSomething
            service.post2SRV($scope, "", sendData, 'true', function(data) {
                $scope.addData(data, param);
            }, function(data) {
                $scope.addData(data, param);
            });
        }

        //处理交易数据，param为分页标识
        $scope.addData = function(msg, param) {
            if (param) {//来自上拉加载,为LoopResult增加分页数据
                var temp = $scope.LoopResult;
                if (msg.LoopResult == null || msg.LoopResult == undefined) {//返回无数据，重置分页
                    param = null;
                }else{
                    $scope.LoopResult = temp.concat(msg.LoopResult);
                }
            } else {//非上拉加载则重置LoopResult
                $scope.LoopResult = null;
                $scope.LoopResult = msg.LoopResult;
            }
            if (msg.LoopResult && msg.LoopResult.length >= 10) {//该次查询记录大于9条，默认还有数据。
                $scope.hasMoreData = true;
            } else {
                //已无再多数据
                $scope.hasMoreData = false;
            }
            if ($scope.LoopResult == null || $scope.LoopResult == undefined) {
                if ("refresh" == scrollFalg) {
                    $scope.$broadcast('scroll.refreshComplete');
                    scrollFalg = null;
                } else if ("infinite" == scrollFalg) {
                    scrollFalg = null;
                    $scope.$broadcast('scroll.infiniteScrollComplete');
                }
                $scope.errorMsg = msg.ResponseMsg;
            }
            if ("refresh" == scrollFalg) {
                $scope.$broadcast('scroll.refreshComplete');
                scrollFalg = null;
            } else if ("infinite" == scrollFalg) {
                scrollFalg = null;
                $scope.$broadcast('scroll.infiniteScrollComplete');
            }
            if (param) {

                $scope.$broadcast('scroll.infiniteScrollComplete');
            }
        }

        /** 上/下拉加载标识 **/
        var scrollFalg = null;
        /** 下拉刷新* */
        $scope.doRefresh = function() {
            scrollFalg = "refresh";
            $scope.errorMsg = null;
            $scope.qryCheck($scope.qryIndex);
            $scope.$broadcast('scroll.refreshComplete');
        }
        // 下拉加载
        $scope.hasMoreData = false;
        // 下拉分页初始值
        $scope.currentPage = 0;
        $scope.loadMore = function() {
            scrollFalg = "infinite";
            $scope.currentPage += 1;// 执行一次页码加1
            var param = parseInt($scope.currentPage) * 10 + 1 + "";
            var indexFlag = $scope.qryIndex;
            var sendData = {
                BeginDate : service.GetDate(0).BeginDate,
                EndDate : service.GetDate(0).EndDate
            }
            if ("0" == indexFlag) {

            } else if ("1" == indexFlag) {
                sendData.BeginDate = service.GetDate(7).BeginDate;
                sendData.EndDate = service.GetDate(7).EndDate;
            } else if ("2" == indexFlag) {
                sendData.BeginDate = service.GetDate(30).BeginDate;
                sendData.EndDate = service.GetDate(30).EndDate;
            } else if ("3" == indexFlag) {
                sendData.BeginDate = $scope.BeginDate;
                sendData.EndDate = $scope.EndDate;
            }
            // 分页开始笔数
            if (param) {
                sendData.BeginNumber = param;
            }
            $scope.QueryAll(sendData, param);
            $scope.$broadcast('scroll.infiniteScrollComplete');
        }
    })
})
